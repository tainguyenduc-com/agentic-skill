---
name: dotnet-ef-migrations
description: "Use when creating, modifying, or generating database migrations. Must use dotnet ef CLI instead of manual SQL scripts."
---

# EF Core Migrations with dotnet ef CLI

## Why dotnet ef, not manual scripts?

The `dotnet ef` CLI is the official Microsoft tool for EF Core migrations. It automatically:
- Generates migration C# code that matches the DbContext model
- Creates `.Designer.cs` snapshot for diff tracking
- Maintains `ModelSnapshot.cs` for future migrations
- Generates idempotent SQL scripts with proper versioning
- Ensures schema changes are compatible with the model

**Never manually write migration scripts.** This causes drift between the C# model and database schema, leading to runtime errors and inconsistencies.

---

## Workflow: Before writing any migration code

### Step 1: Identify the startup project (--startup-project)

The startup project is the entry point that hosts the DbContext. In TaindSoft:

- **App/Web entry points**: `source/apps/taindsoft-app/TaindSoft.IdentityService.App` (or similar)
- **Service hosts**: Any project in `source/hosts/` with a `Program.cs`

To confirm: look for `AddDbContext<YourDbContext>()` in `Program.cs` or dependency injection setup.

Example:
```bash
# For the IdentityService app
--startup-project source/apps/taindsoft-app/TaindSoft.IdentityService.App
```

### Step 2: Identify the target project (--project)

The target project is where migrations are generated. In TaindSoft, this is always the **Infrastructure layer** of the module:

```
source/modules/{ModuleName}/src/{Namespace}.Infrastructure
```

Examples:
- `source/modules/content-management/src/TaindSoft.ContentManagement.Infrastructure`
- `source/modules/customer-management/src/TaindSoft.CustomerManagement.Infrastructure`
- `source/modules/identity-service/src/TaindSoft.IdentityService.Infrastructure`

### Step 3: Identify the DbContext class (--context)

Find the DbContext in the target Infrastructure project. Example locations:
- `{Namespace}.Infrastructure/DbContext.cs` or `{Namespace}.Infrastructure/Persistence/DbContext.cs`
- Fully qualified name: `TaindSoft.ContentManagement.Infrastructure.ContentManagementDbContext`

---

## Common dotnet ef commands

### 1. Add a new migration

After modifying the DbContext model (adding/removing/changing entities/properties):

```bash
dotnet ef migrations add {MigrationName} \
  --project source/modules/{ModuleName}/src/{Namespace}.Infrastructure \
  --startup-project source/apps/taindsoft-app/{App}.csproj \
  --context {DbContextClassName}
```

**Example:**
```bash
dotnet ef migrations add AddProductTable \
  --project source/modules/content-management/src/TaindSoft.ContentManagement.Infrastructure \
  --startup-project source/apps/taindsoft-app/TaindSoft.IdentityService.App \
  --context ContentManagementDbContext
```

**Output:** Two new files in `{Infrastructure}/Migrations/`:
- `{Timestamp}_{MigrationName}.cs` — the C# migration code
- `{Timestamp}_{MigrationName}.Designer.cs` — auto-generated designer file

Do **not** edit these manually unless you know exactly why.

### 2. Remove the last migration (before applying to database)

If you need to undo the latest migration that hasn't been applied yet:

```bash
dotnet ef migrations remove \
  --project source/modules/{ModuleName}/src/{Namespace}.Infrastructure \
  --startup-project source/apps/taindsoft-app/{App}.csproj
```

### 3. Generate a SQL script from migrations

When you need a SQL script to apply the migration to the database:

```bash
dotnet ef migrations script {FromMigration} {ToMigration} \
  --output {--startup-project}/Scripts/{module-alias}/{migrationName}.sql \
  --project source/modules/{ModuleName}/src/{Namespace}.Infrastructure \
  --startup-project source/apps/taindsoft-app/{App}.csproj \
  --idempotent
```

**Example:**
```bash
dotnet ef migrations script 0 AddProductTable \
  --output source/apps/taindsoft-app/Scripts/content-mgmt/20261009_AddProductTable.sql \
  --project source/modules/content-management/src/TaindSoft.ContentManagement.Infrastructure \
  --startup-project source/apps/taindsoft-app/TaindSoft.IdentityService.App \
  --idempotent
```

**Flags explained:**
- `--idempotent`: Script can be run multiple times safely (idempotent SQL with IF NOT EXISTS checks)
- `--output`: Where to save the `.sql` file. Path is relative to the working directory.
- `0`: Start from the initial state (before the first migration)

### 4. List all migrations

See what migrations exist and their status:

```bash
dotnet ef migrations list \
  --project source/modules/{ModuleName}/src/{Namespace}.Infrastructure \
  --startup-project source/apps/taindsoft-app/{App}.csproj
```

---

## Database deployment with DbUp

**IMPORTANT:** TaindSoft uses **DbUp** to apply migrations to the database, NOT `dotnet ef database update`.

Your responsibility ends after:
1. Generating the C# migration files (via `dotnet ef migrations add`)
2. Generating the SQL script (via `dotnet ef migrations script --idempotent`)

DbUp will automatically handle applying the SQL scripts to the database at application runtime. **Never run `dotnet ef database update`** — this is completely redundant and bypasses the project's established deployment workflow.

---

## Checklist before submitting

When implementing a migration task:

1. **Verify the correct host** — confirm which app/service hosts the DbContext (use `--startup-project`)
2. **Verify the Infrastructure project path** — migrations always go in the module's Infrastructure layer
3. **Check DbContext name** — pass the exact class name with `--context`
4. **Add migration first** — `dotnet ef migrations add` to generate the C# code
5. **Review the generated migration** — check `{Timestamp}_{MigrationName}.cs` for correctness
6. **Generate SQL script** — use `dotnet ef migrations script --idempotent` (mandatory)
7. **Never manually edit** the generated `.Designer.cs` or edit the migration C# code unless absolutely necessary
8. **Always commit** both C# migration files AND the SQL script to source control
9. **DO NOT run `dotnet ef database update`** — DbUp handles database deployment automatically

---

## When to NOT use dotnet ef

- **Seeding business data**: Use separate DbUp scripts or application logic; never embed in migrations
- **Direct database updates**: Let DbUp handle execution; don't use `dotnet ef database update`

For all model/schema changes, use `dotnet ef migrations add` + `dotnet ef migrations script`. If unsure, ask.

---

## Troubleshooting

### "No DbContext found"
- Ensure the startup project has `AddDbContext<YourDbContext>()` in `Program.cs`
- Pass `--context` if there are multiple DbContext classes

### "Migrations namespace mismatch"
- Use `--namespace {TargetNamespace}` when creating migrations in non-standard locations

### "Connection string not found"
- Ensure the startup project has valid connection strings in `appsettings.json` or `User Secrets`
- Use `--connection` to override (not recommended for sensitive data)

### "Target runtime mismatch"
- Ensure the startup project targets the same .NET version as the Infrastructure project (typically .NET 10)

---

## References

- [EF Core CLI Tools — Microsoft Learn](https://learn.microsoft.com/en-us/ef/core/cli/dotnet)
- Project structure: `source/modules/{Module}/src/{Namespace}.Infrastructure/Migrations/`
- DbContext pattern: `BaseDbContext` in TaindSoft.Core.Infrastructure
