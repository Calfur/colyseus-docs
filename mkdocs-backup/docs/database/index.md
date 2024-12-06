# Database & Persistance

Colyseus is agnostic to the database you use. You can use your preferred Node.js tool for working with databases.

!!! Note "We're considering implementing our own database tool: `@colyseus/database`"
    We'd love to ship Colyseus 1.0 with our own database solution. We've been experimenting with our own tool built on top of [Kysely](https://kysely.dev/). If you're interested in helping/contributing, please [join the discussion](https://github.com/colyseus/colyseus/issues/594) and let us know!

---

## Recommended database tools

You can use your preferred Node.js tool for working with databases. Query builders are known for their simplicity and flexibility, while ORMs are known for abstracting the database layer and providing a more object-oriented approach.

### ORMs (Object-Relational Mappers)

- [DrizzleORM](https://orm.drizzle.team/)
- [MikroORM](https://mikro-orm.io/)
- [Sequelize](https://sequelize.org/)
- [TypeORM](https://typeorm.io/)
- [Prisma](http://www.prisma.io/)

### Query builders

- [Kysely](https://kysely.dev/)
- [Knex](https://knexjs.org/)
