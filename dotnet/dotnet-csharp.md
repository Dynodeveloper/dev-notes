# 🔷 .NET & C# — Referencia rápida

---

## ⚡ CLI de .NET

```bash
# Crear proyectos
dotnet new console -n MiApp          # App de consola
dotnet new webapi -n MiApi           # Web API
dotnet new classlib -n MiLibreria    # Librería de clases
dotnet new maui -n MiAppMaui         # App MAUI

# Correr y compilar
dotnet run                           # Ejecutar proyecto
dotnet build                         # Compilar
dotnet publish                       # Publicar

# Paquetes NuGet
dotnet add package NombreDelPaquete  # Instalar paquete
dotnet remove package NombreDelPaquete
dotnet restore                       # Restaurar dependencias
dotnet list package                  # Ver paquetes instalados

# Migraciones (Entity Framework)
dotnet ef migrations add NombreMigracion
dotnet ef database update
dotnet ef migrations remove
```

---

## 📦 Namespace y estructura base

```csharp
namespace MiApp.Models  // agrupa clases relacionadas
{
    public class Persona  // clase dentro del namespace
    {
    }
}

// Importar namespace
using MiApp.Models;
```

---

## 🏗️ Clase

```csharp
namespace MiApp.Models
{
    public class Persona
    {
        // Propiedades
        public int Id { get; set; }
        public string Nombre { get; set; } = string.Empty;
        public string? Apellido { get; set; }  // nullable con ?

        // Constructor vacío
        public Persona() { }

        // Constructor con parámetros
        public Persona(int id, string nombre)
        {
            Id = id;
            Nombre = nombre;
        }

        // Método
        public string ObtenerNombreCompleto()
        {
            return $"{Nombre} {Apellido}";
        }
    }
}
```

---

## 🔌 Interface

```csharp
namespace MiApp.Interfaces
{
    public interface IPersonaRepository
    {
        Task<List<Persona>> GetAllAsync();
        Task<Persona?> GetByIdAsync(int id);
        Task AddAsync(Persona persona);
        Task UpdateAsync(Persona persona);
        Task DeleteAsync(int id);
    }
}
```

---

## 🗄️ Repositorio

```csharp
using MiApp.Interfaces;
using MiApp.Models;

namespace MiApp.Repositories
{
    public class PersonaRepository : IPersonaRepository
    {
        private readonly AppDbContext _context;

        // Inyección de dependencia por constructor
        public PersonaRepository(AppDbContext context)
        {
            _context = context;
        }

        public async Task<List<Persona>> GetAllAsync()
        {
            return await _context.Personas.ToListAsync();
        }

        public async Task<Persona?> GetByIdAsync(int id)
        {
            return await _context.Personas.FindAsync(id);
        }

        public async Task AddAsync(Persona persona)
        {
            await _context.Personas.AddAsync(persona);
            await _context.SaveChangesAsync();
        }

        public async Task UpdateAsync(Persona persona)
        {
            _context.Personas.Update(persona);
            await _context.SaveChangesAsync();
        }

        public async Task DeleteAsync(int id)
        {
            var persona = await GetByIdAsync(id);
            if (persona != null)
            {
                _context.Personas.Remove(persona);
                await _context.SaveChangesAsync();
            }
        }
    }
}
```

---

## 🚀 Program.cs — Dependency Injection

```csharp
using MiApp.Interfaces;
using MiApp.Repositories;

var builder = WebApplication.CreateBuilder(args);

// ── Servicios ──────────────────────────────────────

// Repositorios
builder.Services.AddScoped<IPersonaRepository, PersonaRepository>();

// Entity Framework
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

// Controllers
builder.Services.AddControllers();

// Swagger
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// CORS
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowAll", policy =>
        policy.AllowAnyOrigin().AllowAnyMethod().AllowAnyHeader());
});

// ── App ────────────────────────────────────────────

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseCors("AllowAll");
app.UseAuthorization();
app.MapControllers();
app.Run();
```

---

## 🔁 Estructuras de control

```csharp
// ── IF ─────────────────────────────────────────────
if (edad >= 18)
{
    Console.WriteLine("Mayor de edad");
}
else if (edad >= 13)
{
    Console.WriteLine("Adolescente");
}
else
{
    Console.WriteLine("Menor");
}

// If ternario
string resultado = edad >= 18 ? "Mayor" : "Menor";

// ── FOR ────────────────────────────────────────────
for (int i = 0; i < 10; i++)
{
    Console.WriteLine(i);
}

// ── FOREACH ────────────────────────────────────────
List<string> nombres = new List<string> { "Ana", "Luis", "María" };
foreach (var nombre in nombres)
{
    Console.WriteLine(nombre);
}

// ── WHILE ──────────────────────────────────────────
int contador = 0;
while (contador < 5)
{
    Console.WriteLine(contador);
    contador++;
}

// ── DO WHILE ───────────────────────────────────────
do
{
    Console.WriteLine(contador);
    contador++;
} while (contador < 5);

// ── SWITCH ─────────────────────────────────────────
switch (dia)
{
    case "Lunes":
        Console.WriteLine("Inicio de semana");
        break;
    case "Viernes":
        Console.WriteLine("Por fin viernes");
        break;
    default:
        Console.WriteLine("Día normal");
        break;
}

// Switch expression (moderno)
string mensaje = dia switch
{
    "Lunes" => "Inicio de semana",
    "Viernes" => "Por fin viernes",
    _ => "Día normal"
};
```

---

## 🛠️ Funciones / Métodos

```csharp
// Método básico
public void Saludar()
{
    Console.WriteLine("Hola!");
}

// Con parámetros y retorno
public int Sumar(int a, int b)
{
    return a + b;
}

// Async / Await
public async Task<List<Persona>> ObtenerPersonasAsync()
{
    return await _repository.GetAllAsync();
}

// Con valor por defecto
public void Imprimir(string mensaje, bool mayusculas = false)
{
    Console.WriteLine(mayusculas ? mensaje.ToUpper() : mensaje);
}

// Expression body (una línea)
public string Saludar(string nombre) => $"Hola, {nombre}!";
```

---

## 📋 Tipos de datos comunes

```csharp
// Básicos
int numero = 10;
double decimal1 = 3.14;
string texto = "Hola";
bool activo = true;
char letra = 'A';

// Nullable
int? numeroNullable = null;
string? textoNullable = null;

// Colecciones
List<string> lista = new List<string>();
Dictionary<string, int> diccionario = new Dictionary<string, int>();
string[] arreglo = new string[5];

// Var (inferencia de tipo)
var nombre = "Carlos";  // inferido como string
```

---

## 💡 Tips útiles

- `??` operador null-coalescing: `string nombre = texto ?? "Sin nombre";`
- `?.` operador null-conditional: `int? largo = texto?.Length;`
- `$""` string interpolation: `$"Hola {nombre}"`
- `nameof()` para evitar strings mágicos: `nameof(Persona.Nombre)`
- Siempre usar `async/await` en operaciones de base de datos

---

*Actualizado: Marzo 2026*
