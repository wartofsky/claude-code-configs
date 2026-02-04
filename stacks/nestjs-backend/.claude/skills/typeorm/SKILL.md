---
name: typeorm
description: TypeORM patterns for NestJS including entities, repositories, relations, migrations, transactions, and query building. Triggers on TypeORM, entity, repository, migration, database, query, relation, column keywords.
---

# TypeORM Patterns for NestJS

## Setup

### Data Source Configuration

```typescript
// src/database/data-source.ts (for CLI migrations)
import { DataSource } from 'typeorm';
import * as dotenv from 'dotenv';
dotenv.config();

export default new DataSource({
  type: 'postgres',
  host: process.env.DB_HOST ?? 'localhost',
  port: parseInt(process.env.DB_PORT, 10) ?? 5432,
  username: process.env.DB_USERNAME,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  entities: ['src/**/*.entity.ts'],
  migrations: ['src/database/migrations/*.ts'],
  synchronize: false, // NEVER true in production
});
```

### NestJS Module Registration

```typescript
// app.module.ts
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [
    TypeOrmModule.forRootAsync({
      imports: [ConfigModule],
      useFactory: (config: ConfigService) => ({
        type: 'postgres',
        host: config.get('DB_HOST'),
        port: config.get<number>('DB_PORT'),
        username: config.get('DB_USERNAME'),
        password: config.get('DB_PASSWORD'),
        database: config.get('DB_NAME'),
        autoLoadEntities: true,  // Auto-detect entities registered via forFeature
        synchronize: false,
        logging: config.get('NODE_ENV') === 'development',
      }),
      inject: [ConfigService],
    }),
  ],
})
export class AppModule {}

// Feature module
@Module({
  imports: [TypeOrmModule.forFeature([UserEntity, ProfileEntity])],
  providers: [UsersService],
  controllers: [UsersController],
})
export class UsersModule {}
```

## Entity Definition

### Base Entity

```typescript
// src/common/entities/base.entity.ts
import {
  PrimaryGeneratedColumn, CreateDateColumn, UpdateDateColumn, DeleteDateColumn,
} from 'typeorm';

export abstract class BaseEntity {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @CreateDateColumn({ name: 'created_at' })
  createdAt: Date;

  @UpdateDateColumn({ name: 'updated_at' })
  updatedAt: Date;

  @DeleteDateColumn({ name: 'deleted_at' })
  deletedAt: Date;
}
```

### Full Entity Example

```typescript
import { Entity, Column, Index, ManyToOne, OneToMany, ManyToMany,
         JoinColumn, JoinTable, Unique } from 'typeorm';
import { BaseEntity } from '../../common/entities/base.entity';

@Entity('users')
@Unique(['email'])
export class UserEntity extends BaseEntity {
  @Column({ length: 255 })
  @Index()
  email: string;

  @Column({ name: 'first_name', length: 100 })
  firstName: string;

  @Column({ name: 'last_name', length: 100, nullable: true })
  lastName: string;

  @Column({ select: false }) // Excluded from default queries
  password: string;

  @Column({ type: 'enum', enum: Role, default: Role.User })
  role: Role;

  @Column({ default: true, name: 'is_active' })
  isActive: boolean;

  @Column({ type: 'jsonb', nullable: true })
  metadata: Record<string, any>;

  // Relations
  @OneToMany(() => OrderEntity, (order) => order.user)
  orders: OrderEntity[];

  @ManyToOne(() => CompanyEntity, (company) => company.users)
  @JoinColumn({ name: 'company_id' })
  company: CompanyEntity;

  @Column({ name: 'company_id', nullable: true })
  companyId: string;

  @ManyToMany(() => RoleEntity)
  @JoinTable({
    name: 'user_roles',
    joinColumn: { name: 'user_id' },
    inverseJoinColumn: { name: 'role_id' },
  })
  roles: RoleEntity[];
}
```

### Column Types Reference

| TypeScript | PostgreSQL | Decorator |
|-----------|-----------|-----------|
| `string` | `varchar(255)` | `@Column()` |
| `string` | `text` | `@Column('text')` |
| `number` | `integer` | `@Column('int')` |
| `number` | `decimal` | `@Column('decimal', { precision: 10, scale: 2 })` |
| `boolean` | `boolean` | `@Column()` |
| `Date` | `timestamp` | `@Column('timestamp')` |
| `object` | `jsonb` | `@Column('jsonb')` |
| `enum` | `enum` | `@Column({ type: 'enum', enum: MyEnum })` |
| `string[]` | `text[]` | `@Column('text', { array: true })` |

## CRUD Operations

### Repository Pattern in Service

```typescript
@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(UserEntity)
    private readonly usersRepo: Repository<UserEntity>,
  ) {}

  // CREATE
  async create(dto: CreateUserDto): Promise<UserEntity> {
    const user = this.usersRepo.create(dto);
    return this.usersRepo.save(user);
  }

  // READ - Single
  async findOne(id: string): Promise<UserEntity> {
    const user = await this.usersRepo.findOne({
      where: { id },
      relations: ['company', 'roles'],
    });
    if (!user) throw new NotFoundException(`User #${id} not found`);
    return user;
  }

  // READ - List with pagination
  async findAll(query: PaginationDto): Promise<[UserEntity[], number]> {
    return this.usersRepo.findAndCount({
      where: { isActive: true },
      relations: ['company'],
      select: ['id', 'email', 'firstName', 'lastName', 'role', 'createdAt'],
      skip: (query.page - 1) * query.limit,
      take: query.limit,
      order: { createdAt: 'DESC' },
    });
  }

  // UPDATE
  async update(id: string, dto: UpdateUserDto): Promise<UserEntity> {
    const user = await this.findOne(id);
    Object.assign(user, dto);
    return this.usersRepo.save(user);
  }

  // DELETE (soft)
  async remove(id: string): Promise<void> {
    const user = await this.findOne(id);
    await this.usersRepo.softRemove(user);
  }

  // UPSERT
  async upsert(dto: CreateUserDto): Promise<UserEntity> {
    await this.usersRepo.upsert(dto, ['email']);
    return this.usersRepo.findOneBy({ email: dto.email });
  }
}
```

## QueryBuilder

For complex queries that `find()` options can't handle.

```typescript
// Filtered list with search, sort, and pagination
async findFiltered(filters: FilterDto): Promise<[UserEntity[], number]> {
  const qb = this.usersRepo.createQueryBuilder('user')
    .leftJoinAndSelect('user.company', 'company');

  // Conditional filters
  if (filters.search) {
    qb.andWhere(
      '(user.firstName ILIKE :search OR user.lastName ILIKE :search OR user.email ILIKE :search)',
      { search: `%${filters.search}%` },
    );
  }

  if (filters.role) {
    qb.andWhere('user.role = :role', { role: filters.role });
  }

  if (filters.companyId) {
    qb.andWhere('user.companyId = :companyId', { companyId: filters.companyId });
  }

  if (filters.isActive !== undefined) {
    qb.andWhere('user.isActive = :isActive', { isActive: filters.isActive });
  }

  // Sort
  const sortField = filters.sortBy ?? 'createdAt';
  const sortOrder = filters.sortOrder ?? 'DESC';
  qb.orderBy(`user.${sortField}`, sortOrder);

  // Pagination
  qb.skip((filters.page - 1) * filters.limit).take(filters.limit);

  return qb.getManyAndCount();
}

// Aggregations
async getOrderStats(userId: string) {
  return this.ordersRepo
    .createQueryBuilder('order')
    .select('order.status', 'status')
    .addSelect('COUNT(*)', 'count')
    .addSelect('SUM(order.total)', 'totalAmount')
    .where('order.userId = :userId', { userId })
    .groupBy('order.status')
    .getRawMany();
}

// Subquery
async findUsersWithOrders() {
  const subQuery = this.ordersRepo
    .createQueryBuilder('order')
    .select('order.userId')
    .where('order.status = :status', { status: 'completed' });

  return this.usersRepo
    .createQueryBuilder('user')
    .where(`user.id IN (${subQuery.getQuery()})`)
    .setParameters(subQuery.getParameters())
    .getMany();
}
```

## Relations

### One-to-Many / Many-to-One

```typescript
// User has many Orders
@Entity('users')
export class UserEntity {
  @OneToMany(() => OrderEntity, (order) => order.user)
  orders: OrderEntity[];
}

@Entity('orders')
export class OrderEntity {
  @ManyToOne(() => UserEntity, (user) => user.orders, { onDelete: 'CASCADE' })
  @JoinColumn({ name: 'user_id' })
  user: UserEntity;

  @Column({ name: 'user_id' })
  userId: string;
}
```

### Many-to-Many

```typescript
@Entity('users')
export class UserEntity {
  @ManyToMany(() => RoleEntity, { eager: false })
  @JoinTable({ name: 'user_roles' })
  roles: RoleEntity[];
}

// With custom join table entity (for extra columns)
@Entity('user_roles')
export class UserRoleEntity extends BaseEntity {
  @ManyToOne(() => UserEntity, { onDelete: 'CASCADE' })
  @JoinColumn({ name: 'user_id' })
  user: UserEntity;

  @ManyToOne(() => RoleEntity, { onDelete: 'CASCADE' })
  @JoinColumn({ name: 'role_id' })
  role: RoleEntity;

  @Column({ name: 'assigned_at', type: 'timestamp', default: () => 'NOW()' })
  assignedAt: Date;
}
```

### One-to-One

```typescript
@Entity('users')
export class UserEntity {
  @OneToOne(() => ProfileEntity, (profile) => profile.user, { cascade: true })
  profile: ProfileEntity;
}

@Entity('profiles')
export class ProfileEntity {
  @OneToOne(() => UserEntity, (user) => user.profile)
  @JoinColumn({ name: 'user_id' })
  user: UserEntity;
}
```

## Transactions

```typescript
import { DataSource } from 'typeorm';

@Injectable()
export class OrdersService {
  constructor(private readonly dataSource: DataSource) {}

  async createOrder(dto: CreateOrderDto, userId: string): Promise<OrderEntity> {
    return this.dataSource.transaction(async (manager) => {
      // Create order
      const order = manager.create(OrderEntity, {
        userId,
        status: OrderStatus.Pending,
        total: 0,
      });
      await manager.save(order);

      // Create order items and calculate total
      let total = 0;
      for (const item of dto.items) {
        const product = await manager.findOneBy(ProductEntity, { id: item.productId });
        if (!product) throw new NotFoundException(`Product #${item.productId} not found`);

        const orderItem = manager.create(OrderItemEntity, {
          orderId: order.id,
          productId: product.id,
          quantity: item.quantity,
          price: product.price,
        });
        await manager.save(orderItem);
        total += product.price * item.quantity;

        // Update inventory
        await manager.decrement(ProductEntity, { id: product.id }, 'stock', item.quantity);
      }

      // Update order total
      order.total = total;
      return manager.save(order);
    });
  }
}
```

## Migrations

```bash
# Generate migration from entity changes
npx typeorm-ts-node-commonjs migration:generate src/database/migrations/AddUserRole -d src/database/data-source.ts

# Create empty migration
npx typeorm-ts-node-commonjs migration:create src/database/migrations/SeedAdminUser

# Run migrations
npx typeorm-ts-node-commonjs migration:run -d src/database/data-source.ts

# Revert last migration
npx typeorm-ts-node-commonjs migration:revert -d src/database/data-source.ts
```

### Migration Example

```typescript
import { MigrationInterface, QueryRunner, Table, TableIndex } from 'typeorm';

export class CreateUsersTable1700000000000 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.createTable(
      new Table({
        name: 'users',
        columns: [
          { name: 'id', type: 'uuid', isPrimary: true, generationStrategy: 'uuid', default: 'uuid_generate_v4()' },
          { name: 'email', type: 'varchar', length: '255', isUnique: true },
          { name: 'first_name', type: 'varchar', length: '100' },
          { name: 'last_name', type: 'varchar', length: '100', isNullable: true },
          { name: 'password', type: 'varchar' },
          { name: 'role', type: 'enum', enum: ['user', 'admin'], default: `'user'` },
          { name: 'is_active', type: 'boolean', default: true },
          { name: 'created_at', type: 'timestamp', default: 'NOW()' },
          { name: 'updated_at', type: 'timestamp', default: 'NOW()' },
          { name: 'deleted_at', type: 'timestamp', isNullable: true },
        ],
      }),
      true,
    );

    await queryRunner.createIndex('users', new TableIndex({
      name: 'IDX_users_email',
      columnNames: ['email'],
    }));
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.dropTable('users');
  }
}
```

## Indexes & Performance

```typescript
// Single column index
@Index()
@Column()
email: string;

// Named index
@Index('IDX_user_email')
@Column()
email: string;

// Composite index (on entity)
@Entity('orders')
@Index(['userId', 'status'])
@Index(['createdAt'])
export class OrderEntity extends BaseEntity { /* ... */ }

// Unique index
@Index('UQ_user_email', { unique: true })
@Column()
email: string;

// Partial index (PostgreSQL)
// Use migration for partial indexes:
// CREATE INDEX idx_active_users ON users(email) WHERE is_active = true;
```

## Best Practices

1. **Always use migrations** — never `synchronize: true` in production
2. **Use `findAndCount`** for paginated queries
3. **Explicit relations** — use `relations` option or QueryBuilder joins, avoid lazy loading
4. **Select needed columns** — don't fetch entire entities for lists
5. **Index foreign keys** — TypeORM doesn't auto-index FK columns
6. **Soft delete** — use `@DeleteDateColumn` and `softRemove()`
7. **Transactions** — wrap multi-step writes in `dataSource.transaction()`
8. **Parameterized queries** — never interpolate values into raw SQL
9. **Use UUIDs** — prefer `uuid` over auto-increment for primary keys
10. **Base entity** — share `id`, `createdAt`, `updatedAt`, `deletedAt` via inheritance
