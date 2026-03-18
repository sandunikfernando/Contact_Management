app/state.rs
AppState now holds a SeaORM `DatabaseConnection` instead of in-memory repos.

Why DatabaseConnection instead of a pool?
SeaORM's DatabaseConnection already IS a connection pool internally (backed by sqlx's PgPool).  You configure pool size via ConnectOptions, and SeaORM manages acquiring/releasing connections automatically.

DatabaseConnection implements Clone cheaply — it's just a reference-counted handle to the pool — so every Axum handler receives its own clone of the handle, all pointing at the same underlying pool.


entity/persons.rs
ActiveModelBehavior lets you hook into before_save / after_save lifecycle events.  We don't need custom logic, so we use the default impl.


target
by running cargo build, the target file generate automatically



-- migrations/001_init.sql
-- Run this against your PostgreSQL database before starting the server.
--
-- Table design notes:
--   - SERIAL      → auto-incrementing integer PK (PostgreSQL specific)
--   - NOT NULL    → enforced at DB level (SeaORM also validates in Rust)
--   - ON DELETE CASCADE → if a person is deleted, their mobiles/emails
--                         are automatically removed by the DB as a safety net
--                         (the service layer also does this, but DB-level
--                          cascade is a good safety backstop)