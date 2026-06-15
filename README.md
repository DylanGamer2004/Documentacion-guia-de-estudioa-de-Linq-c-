# Manual Completo de LINQ en C#

[![.NET](https://img.shields.io/badge/.NET-8.0-512BD4?style=for-the-badge&logo=dotnet)](https://dotnet.microsoft.com)
[![C#](https://img.shields.io/badge/C%23-12-239120?style=for-the-badge&logo=csharp)](https://docs.microsoft.com/dotnet/csharp/)
[![SQL Server](https://img.shields.io/badge/SQL%20Server-2022-CC2927?style=for-the-badge&logo=microsoftsqlserver)](https://www.microsoft.com/sql-server)
[![EF Core](https://img.shields.io/badge/EF%20Core-8.0-5C2D91?style=for-the-badge)](https://docs.microsoft.com/ef/core/)
[![License](https://img.shields.io/badge/Licencia-MIT-green?style=for-the-badge)](LICENSE)

> **Language Integrated Query** — Consultas integradas directamente en el lenguaje C#  
> **Guia de referencia completa** | .NET 8+ | C# 12 | Visual Studio 2022 | SQL Server | Arquitectura 4 Capas | GitHub Actions

---

## Tabla de Contenidos

| # | Seccion | Temas Clave |
|---|---------|-------------|
| 01 | [Que es LINQ?](#1-que-es-linq) | Origen, ventajas, antes/despues, arquitectura interna |
| 02 | [Tipos de LINQ](#2-tipos-de-linq) | Objects, SQL, XML, Entities, DataSet, PLINQ |
| 03 | [Sintaxis: Query vs Method](#3-sintaxis-query-vs-method) | Comparativa, cuando usar cada una, conversion |
| 04 | [Operadores Estandar](#4-operadores-estandar) | Filtrado, proyeccion, ordenamiento, agrupacion, joins |
| 05 | [Funciones de Agregado — Guia Completa](#5-funciones-de-agregado--guia-completa) | Count, Sum, Average, Min, Max, MinBy, MaxBy, Aggregate, estadisticas |
| 06 | [LINQ con Colecciones](#6-linq-con-colecciones) | List, Array, Dictionary, String, tipos anonimos, subqueries |
| 07 | [LINQ con Entity Framework](#7-linq-con-entity-framework) | DbContext, Include, AsNoTracking, proyeccion, raw SQL, SPs |
| 08 | [LINQ Async — Patrones Asincronos](#8-linq-async--patrones-asincronos) | ToListAsync, CountAsync, IAsyncEnumerable, StreamAsync |
| 09 | [LINQ to XML y JSON](#9-linq-to-xml-y-json) | XDocument, XElement, System.Text.Json, transformaciones |
| 10 | [Expresiones Lambda y Arboles de Expresion](#10-expresiones-lambda-y-arboles-de-expresion) | Func, Action, Expression, PredicateBuilder, dinamico |
| 11 | [Ejecucion Diferida vs Inmediata](#11-ejecucion-diferida-vs-inmediata) | Streaming, non-streaming, materializacion, trampas |
| 12 | [PLINQ — LINQ Paralelo](#12-plinq--linq-paralelo) | AsParallel, AsOrdered, cancelacion, ForAll, rendimiento |
| 13 | [Arquitectura de 4 Capas con LINQ](#13-arquitectura-de-4-capas-con-linq) | Capas, repositorio, servicio, presentacion, DI |
| 14 | [Proyecto Integrado — CRUD Completo](#14-proyecto-integrado--crud-completo) | Solucion completa, migraciones, seeding, ejecucion |
| 15 | [Patrones Avanzados y Operadores Custom](#15-patrones-avanzados-y-operadores-custom) | Extension methods, Pivot, Chunk, paginacion generica |
| 16 | [Casos de Uso Avanzados](#16-casos-de-uso-avanzados) | Filtros dinamicos, reportes, pipelines de datos |
| 17 | [LINQ en GitHub — Repositorio de Ejemplo](#17-linq-en-github--repositorio-de-ejemplo) | Estructura, CI/CD, pruebas, .gitignore |
| 18 | [Rendimiento y Optimizacion](#18-rendimiento-y-optimizacion) | Profiling, SQL generado, indices, AsSplitQuery |
| 19 | [Errores Comunes y Buenas Practicas](#19-errores-comunes-y-buenas-practicas) | N+1, memoria, async, tracking, proyeccion |
| 20 | [De SQL a LINQ — Guia de Migracion](#20-de-sql-a-linq--guia-de-migracion) | Tabla de equivalencias, patrones traducidos |
| 21 | [Ejercicios y Desafios](#21-ejercicios-y-desafios) | 15 ejercicios progresivos con soluciones |
| 22 | [Referencia Rapida](#22-referencia-rapida) | Todos los operadores, firmas, ejemplos de una linea |

---

## 1. Que es LINQ?

**LINQ** (*Language Integrated Query*) es una caracteristica de C# introducida en **.NET 3.5 (2007)** que permite realizar consultas sobre colecciones de datos utilizando una sintaxis integrada directamente en el lenguaje. Antes de LINQ, cada tipo de fuente de datos requeria una API diferente: SQL para bases de datos, XPath para XML, bucles `foreach` para colecciones en memoria. LINQ unifica todo esto bajo un modelo de programacion unico y consistente.

La arquitectura interna de LINQ se basa en dos interfaces fundamentales: `IEnumerable<T>` para consultas en memoria (LINQ to Objects) y `IQueryable<T>` para consultas que se traducen a otro lenguaje como SQL. Cuando escribes una consulta LINQ contra un `IQueryable<T>`, los operadores no ejecutan codigo directamente, sino que construyen un **arbol de expresion** que el proveedor (como Entity Framework) traduce al lenguaje destino.

### Antes y Despues de LINQ

```csharp
// ❌ SIN LINQ — Imperativo, verboso, propenso a errores
var resultado = new List<string>();
foreach (var nombre in nombres)
{
    if (nombre.StartsWith("A"))
        resultado.Add(nombre);
}
resultado.Sort();

// ✅ CON LINQ — Declarativo, limpio, tipado
var resultado = nombres
    .Where(n => n.StartsWith("A"))
    .OrderBy(n => n)
    .ToList();
```

### Arquitectura Interna de LINQ

```
                    Codigo C#
                       │
            ┌──────────▼──────────┐
            │   Compilador C#     │
            │  (traduce a         │
            │   llamadas de       │
            │   metodos extension)│
            └──────────┬──────────┘
                       │
          ┌────────────▼────────────┐
          │   Tipo de Secuencia     │
          │                         │
    ┌─────▼─────┐           ┌──────▼──────┐
    │IEnumerable│           │ IQueryable  │
    │   <T>     │           │    <T>      │
    └─────┬─────┘           └──────┬──────┘
          │                        │
    ┌─────▼──────┐          ┌─────▼──────┐
    │  LINQ to   │          │  Proveedor │
    │  Objects   │          │  (EF Core) │
    │(en memoria)│          │(traduce a  │
    └─────┬──────┘          │   SQL)     │
          │                 └─────┬──────┘
    ┌─────▼──────┐          ┌─────▼──────┐
    │  IL Code   │          │  SQL Server │
    │  (directo) │          │  (remoto)   │
    └────────────┘          └────────────┘
```

### Ventajas Clave de LINQ

| Ventaja | Descripcion | Ejemplo |
|---------|-------------|---------|
| **Legibilidad** | Código declarativo, expresa "que" no "como" | `.Where(p => p.Activo)` vs bucle foreach |
| **Seguridad de tipos** | Errores detectados en compilacion, no en ejecucion | `p.Precioo` no compila |
| **IntelliSense** | Autocompletado total en Visual Studio/Rider | Todas las propiedades disponibles |
| **Unificacion** | Misma sintaxis para objetos, XML, BD, JSON | `from x in fuente` funciona en cualquier fuente |
| **Composicion** | Operadores encadenables como pipeline | `.Where().OrderBy().Select().Take()` |
| **Depuracion** | Puedes inspeccionar consultas paso a paso | Breakpoint en cada operador |

### Espacios de Names Necesarios

```csharp
using System.Linq;                    // Operadores LINQ core
using System.Collections.Generic;     // IEnumerable<T>, List<T>
using System.Xml.Linq;                // LINQ to XML (XDocument, XElement)
using Microsoft.EntityFrameworkCore;  // EF Core (Include, AsNoTracking, ToListAsync)
using System.Linq.Expressions;        // Expression<T> (arboles de expresion)
using System.Threading.Tasks;         // Async/await
```

---

## 2. Tipos de LINQ

```
LINQ
├── LINQ to Objects     → Listas, arreglos, IEnumerable<T> (en memoria)
├── LINQ to SQL         → SQL Server directo (obsoleto, usar EF Core)
├── LINQ to Entities    → Entity Framework Core (recomendado para BD)
├── LINQ to XML         → XDocument, XElement (crear/consultar/modificar XML)
├── LINQ to DataSet     → ADO.NET DataTable/DataSet (legacy)
├── LINQ to JSON        → System.Text.Json.Nodes (JSON)
└── PLINQ               → Parallel LINQ (multi-hilo, CPU intensivo)
```

### LINQ to Objects — En Memoria

```csharp
int[] numeros = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

var pares = numeros.Where(n => n % 2 == 0).ToList();
// Resultado: [2, 4, 6, 8, 10]

var cuadrados = numeros.Select(n => n * n).ToList();
// Resultado: [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]

var sumaPares = numeros.Where(n => n % 2 == 0).Sum();
// Resultado: 30
```

### LINQ to XML — Documentos XML

```csharp
// Crear documento XML
var doc = new XDocument(
    new XElement("Productos",
        from p in productos
        select new XElement("Producto",
            new XAttribute("Id", p.Id),
            new XElement("Nombre", p.Nombre),
            new XElement("Precio", p.Precio),
            new XElement("Categoria", p.Categoria)
        )
    )
);

// Consultar XML con LINQ
var caros = from p in doc.Descendants("Producto")
            where (decimal)p.Element("Precio") > 100
            select new {
                Id = (int)p.Attribute("Id"),
                Nombre = (string)p.Element("Nombre"),
                Precio = (decimal)p.Element("Precio")
            };

// Modificar XML
doc.Descendants("Producto")
    .Where(p => (string)p.Element("Categoria") == "Computo")
    .ToList()
    .ForEach(p => p.Element("Precio")?.SetValue(
        (decimal)p.Element("Precio") * 0.9m));

// Guardar a archivo
doc.Save("productos.xml");
```

### LINQ to JSON — System.Text.Json

```csharp
using System.Text.Json.Nodes;

// Parsear JSON
string json = """{"productos": [{"nombre": "Laptop", "precio": 1200}, {"nombre": "Mouse", "precio": 35}]}""";
var nodo = JsonNode.Parse(json);

// Consultar con LINQ
var caros = nodo!["productos"]!.AsArray()
    .Where(p => (decimal)p!["precio"]! > 100)
    .Select(p => new {
        Nombre = (string)p!["nombre"]!,
        Precio = (decimal)p!["precio"]!
    });

// Crear JSON con LINQ
var jsonResult = new JsonObject {
    ["resumen"] = new JsonObject {
        ["total"] = productos.Count(),
        ["valorInventario"] = productos.Sum(p => p.Precio * p.Stock),
        ["categorias"] = new JsonArray(
            productos
                .GroupBy(p => p.Categoria)
                .Select(g => (JsonNode)new JsonObject {
                    ["nombre"] = g.Key,
                    ["cantidad"] = g.Count(),
                    ["promedio"] = g.Average(p => p.Precio)
                })
                .ToArray()
        )
    }
};
Console.WriteLine(jsonResult.ToJsonString(new JsonSerializerOptions { WriteIndented = true }));
```

### PLINQ — LINQ Paralelo

```csharp
// Procesa en paralelo usando todos los nucleos del CPU
var resultado = datos.AsParallel()
    .Where(x => x.EsValido())
    .Select(x => x.Procesar())
    .ToList();

// Mantener orden original
var ordenado = datos.AsParallel().AsOrdered()
    .Where(x => x.Valor > 100)
    .Select(x => x.Transformar())
    .ToList();

// Con cancelacion
var cts = new CancellationTokenSource();
var cancelable = datos.AsParallel()
    .WithCancellation(cts.Token)
    .WithDegreeOfParallelism(4)
    .Select(x => Procesar(x));

// ForAll: ejecutar sin recolectar resultados
datos.AsParallel()
    .Where(p => p.Stock < 10)
    .ForAll(p => EnviarAlerta(p));
```

---

## 3. Sintaxis: Query vs Method

LINQ tiene **dos sintaxis equivalentes** — ambas compilan al mismo IL y producen resultados identicos. La sintaxis de metodo (fluent) es mas flexible y cubre todos los operadores, mientras que la sintaxis de query es mas legible para joins y agrupamientos complejos.

### Sintaxis de Query (SQL-like)

```csharp
var consulta = from p in productos
               where p.Precio > 100 && p.Stock > 0
               orderby p.Categoria, p.Precio descending
               select new { p.Nombre, p.Precio, p.Categoria };
```

### Sintaxis de Metodo (Fluent / Lambda)

```csharp
var consulta = productos
    .Where(p => p.Precio > 100 && p.Stock > 0)
    .OrderBy(p => p.Categoria)
    .ThenByDescending(p => p.Precio)
    .Select(p => new { p.Nombre, p.Precio, p.Categoria });
```

### Comparativa Completa

| Operacion | Query Syntax | Method Syntax | Preferida |
|-----------|-------------|---------------|-----------|
| Filtrar | `where p.Activo` | `.Where(p => p.Activo)` | Method |
| Ordenar | `orderby p.Nombre` | `.OrderBy(p => p.Nombre)` | Method |
| Proyectar | `select p.Nombre` | `.Select(p => p.Nombre)` | Method |
| Unir | `join c in clientes on p.Id equals c.Id` | `.Join(clientes, ...)` | Query |
| Agrupar | `group p by p.Categoria into g` | `.GroupBy(p => p.Categoria)` | Query |
| Variable | `let total = p.Precio * p.Stock` | No directo (usar Select anidado) | Query |
| Distinct | No disponible | `.Distinct()` | Method |
| Take/Skip | No disponible | `.Take(5).Skip(10)` | Method |
| Paginacion | No disponible | `.Skip(n).Take(m)` | Method |

> **Recomendacion:** Usa **Method Syntax** como predeterminada — es mas flexible, cubre todos los operadores, y es mas facil de encadenar. Usa **Query Syntax** solo para joins complejos y agrupamientos donde la legibilidad mejore notablemente.

### let en Query Syntax — Variables Intermedias

```csharp
// Query Syntax con let — mas legible
var conDescuento = from p in productos
                   let precioIVA = p.Precio * 1.16m
                   let descuento = precioIVA > 500 ? 0.10m : 0.05m
                   let precioFinal = precioIVA * (1 - descuento)
                   where precioFinal > 50
                   orderby precioFinal descending
                   select new {
                       p.Nombre,
                       PrecioBase = p.Precio,
                       ConIVA = precioIVA,
                       Descuento = descuento * 100 + "%",
                       PrecioFinal = precioFinal
                   };

// Equivalente en Method Syntax — mas verboso
var conDescuento2 = productos
    .Select(p => new { p, precioIVA = p.Precio * 1.16m })
    .Select(x => new { x.p, x.precioIVA, descuento = x.precioIVA > 500 ? 0.10m : 0.05m })
    .Select(x => new { x.p, x.precioIVA, x.descuento, precioFinal = x.precioIVA * (1 - x.descuento) })
    .Where(x => x.precioFinal > 50)
    .OrderByDescending(x => x.precioFinal)
    .Select(x => new {
        x.p.Nombre,
        PrecioBase = x.p.Precio,
        ConIVA = x.precioIVA,
        Descuento = x.descuento * 100 + "%",
        PrecioFinal = x.precioFinal
    });
```

---

## 4. Operadores Estandar

### 4.1 Filtrado

```csharp
var lista = new List<Producto> { /* ... */ };

// Where — filtro basico
var caros = lista.Where(p => p.Precio > 500);

// Where con condiciones multiples
var filtroCompuesto = lista.Where(p =>
    p.Activo && p.Stock > 0 && p.Categoria == "Electronica");

// Where con indice (segundo parametro = posicion del elemento)
var primeros5Caros = lista
    .Where((p, index) => p.Precio > 100 && index < 10);

// Where con operador OR
var busquedaMultiple = lista.Where(p =>
    p.Nombre.Contains("Laptop") || p.Nombre.Contains("PC"));

// OfType — filtra por tipo en colecciones heterogeneas
object[] mixta = { 1, "hola", 3.14, 42, true, "mundo", 7 };
var enteros = mixta.OfType<int>();      // 1, 42, 7
var cadenas = mixta.OfType<string>();    // "hola", "mundo"
```

### 4.2 Proyeccion

```csharp
// Select — transforma cada elemento
var nombres = lista.Select(p => p.Nombre);

// Select con tipo anonimo
var catalogo = lista.Select(p => new {
    p.Id,
    p.Nombre,
    PrecioFormateado = $"${p.Precio:N2}",
    Disponibilidad = p.Stock > 10 ? "Suficiente" : "Poco stock"
});

// Select con indice
var conIndice = lista.Select((p, i) =>
    $"#{i + 1} {p.Nombre} - ${p.Precio:F2}");

// Select a tipo concreto (DTO)
var dtos = lista.Select(p => new ProductoDTO {
    Id = p.Id,
    Nombre = p.Nombre,
    PrecioConIVA = p.Precio * 1.16m,
    Estado = p.Stock > 20 ? "Disponible" :
             p.Stock > 0  ? "Bajo Stock" : "Agotado"
}).ToList();

// SelectMany — aplana colecciones anidadas
var todasEtiquetas = productos.SelectMany(p => p.Etiquetas);

// SelectMany con resultado tipado
var productosConTags = productos
    .SelectMany(p => p.Etiquetas, (producto, tag) => new {
        Producto = producto.Nombre,
        Tag = tag
    });

// SelectMany: pedidos de todos los clientes
var todosLosPedidos = clientes
    .SelectMany(c => c.Pedidos)
    .Where(p => p.Total > 1000)
    .OrderByDescending(p => p.Total);
```

### 4.3 Ordenamiento

```csharp
// Orden ascendente
var porNombre = lista.OrderBy(p => p.Nombre);

// Orden descendente
var porPrecioDesc = lista.OrderByDescending(p => p.Precio);

// Orden multiple (equivalente a ORDER BY Dept, Salario DESC, Nombre)
var multiOrden = lista
    .OrderBy(p => p.Categoria)
    .ThenBy(p => p.Nombre)
    .ThenByDescending(p => p.Precio);

// Ordenamiento con condicion personalizada
var porPrioridad = tareas
    .OrderBy(t => t.Prioridad == "Alta" ? 0 :
                  t.Prioridad == "Media" ? 1 : 2)
    .ThenBy(t => t.FechaLimite);

// Invertir orden
var invertido = lista.Reverse();

// Ordenar por propiedad en runtime (reflection)
var ordenDinamico = lista.OrderBy(p =>
    (string)typeof(Producto).GetProperty("Nombre")!.GetValue(p)!);
```

> **Error comun:** No uses `OrderBy` encadenados para multiples campos. Cada `OrderBy` **resetea** el ordenamiento. Usa `ThenBy` despues del primer `OrderBy`.

### 4.4 Agrupacion

```csharp
// GroupBy — agrupa elementos por clave
var porCategoria = lista.GroupBy(p => p.Categoria);

foreach (var grupo in porCategoria)
{
    Console.WriteLine($"Categoria: {grupo.Key} ({grupo.Count()} productos)");
    foreach (var p in grupo)
        Console.WriteLine($"  - {p.Nombre}: ${p.Precio}");
}

// GroupBy con proyeccion — estadisticas por grupo
var resumen = lista
    .GroupBy(p => p.Categoria)
    .Select(g => new {
        Categoria = g.Key,
        Total = g.Count(),
        PrecioPromedio = g.Average(p => p.Precio),
        PrecioMinimo = g.Min(p => p.Precio),
        PrecioMaximo = g.Max(p => p.Precio),
        StockTotal = g.Sum(p => p.Stock),
        ValorInventario = g.Sum(p => p.Precio * p.Stock)
    })
    .OrderByDescending(x => x.ValorInventario);

// GroupBy con clave compuesta
var porCategoriaRango = lista.GroupBy(p => new {
    p.Categoria,
    Rango = p.Precio >= 500 ? "Premium" : "Estandar"
});

// GroupBy con proyeccion de elementos
var nombresPorCategoria = lista
    .GroupBy(p => p.Categoria, p => p.Nombre);

// ToLookup — agrupacion inmediata (materializada)
var lookup = lista.ToLookup(p => p.Categoria);
if (lookup.Contains("Electronica"))
{
    var electronics = lookup["Electronica"]; // O(1) acceso
}
```

### 4.5 Joins (Uniones)

```csharp
var clientes = new List<Cliente> { /* ... */ };
var pedidos  = new List<Pedido>  { /* ... */ };

// ── Inner Join ──
var resultado = clientes.Join(
    pedidos,
    c => c.Id,           // clave de clientes
    p => p.ClienteId,    // clave de pedidos
    (c, p) => new {      // resultado
        Cliente = c.Nombre,
        Pedido  = p.Numero,
        Total   = p.Total
    });

// Inner Join con Query Syntax
var innerJoin = from c in clientes
                join p in pedidos on c.Id equals p.ClienteId
                select new { c.Nombre, p.Numero, p.Total };

// ── Left Join (GroupJoin + DefaultIfEmpty) ──
var leftJoin = clientes.GroupJoin(
    pedidos,
    c => c.Id,
    p => p.ClienteId,
    (c, pedidosCliente) => new {
        Cliente = c.Nombre,
        Pedidos = pedidosCliente.DefaultIfEmpty()
    });

// Left Join con Query Syntax
var leftJoinQ = from c in clientes
                join p in pedidos on c.Id equals p.ClienteId into gp
                from p in gp.DefaultIfEmpty()
                select new {
                    Cliente = c.Nombre,
                    Pedido = p != null ? p.Numero : "SIN PEDIDO",
                    Total  = p != null ? p.Total : 0
                };

// ── Cross Join ──
var combinaciones = from c in colores
                    from t in tallas
                    select new { Color = c, Talla = t };

// ── Join con clave compuesta ──
var compuesto = ordenes.Join(
    clientes,
    o => new { o.ClienteId, o.Sucursal },
    c => new { c.Id, c.Sucursal },
    (o, c) => new { o.Numero, c.Nombre, o.Total });

// ── Zip — combinar por posicion ──
var nombres = new[] { "Ana", "Carlos", "Maria" };
var salarios = new[] { 3500m, 4200m, 5500m };
var combinados = nombres.Zip(salarios, (nombre, salario) =>
    $"{nombre} gana ${salario:F2}");
```

### 4.6 Paginacion

```csharp
int pagina = 2;
int tamanoPagina = 10;

var paginado = lista
    .OrderBy(p => p.Nombre)         // SIEMPRE ordenar antes de paginar
    .Skip((pagina - 1) * tamanoPagina)
    .Take(tamanoPagina)
    .ToList();

// TakeWhile / SkipWhile
var hastaCien = lista.OrderBy(p => p.Precio)
    .TakeWhile(p => p.Precio < 100);

var despuesBaratos = lista.OrderBy(p => p.Precio)
    .SkipWhile(p => p.Precio < 50);
```

### 4.7 Conjuntos

```csharp
var a = new[] { 1, 2, 3, 4, 5 };
var b = new[] { 3, 4, 5, 6, 7 };

var union        = a.Union(b);        // [1,2,3,4,5,6,7] — sin duplicados
var interseccion = a.Intersect(b);    // [3,4,5]
var diferencia   = a.Except(b);       // [1,2]
var distintos    = a.Distinct();      // elimina duplicados
var concat       = a.Concat(b);       // [1,2,3,4,5,3,4,5,6,7] — con duplicados

// DistinctBy — por propiedad (.NET 6+)
var porCategoria = productos.DistinctBy(p => p.Categoria);

// UnionBy — por propiedad (.NET 6+)
var unionPorNombre = listaA.UnionBy(listaB, p => p.Nombre);

// IntersectBy / ExceptBy — con selector de clave (.NET 6+)
var idsComunes = listaA.Select(p => p.Id).IntersectBy(listaB.Select(p => p.Id), id => id);

// Con IEqualityComparer personalizado
var sinDuplicadosNombre = productos.DistinctBy(p => p.Nombre, StringComparer.OrdinalIgnoreCase);
```

### 4.8 Elementos Individuales

```csharp
// Primer elemento (lanza excepcion si vacio)
var primero = lista.First();
var primerActivo = lista.First(p => p.Activo);

// Primer elemento o null
var primeroONull = lista.FirstOrDefault();
var primeroONull2 = lista.FirstOrDefault(p => p.Precio > 1000);

// Ultimo elemento
var ultimo = lista.Last();
var ultimoONull = lista.LastOrDefault();

// Elemento unico (lanza si hay mas de uno o ninguno)
var unico = lista.Single(p => p.Id == 42);
var unicoONull = lista.SingleOrDefault(p => p.Id == 42);

// Por indice
var tercero = lista.ElementAt(2);
var terceroONull = lista.ElementAtOrDefault(200); // null si fuera de rango
```

### 4.9 Cuantificadores y Verificadores

```csharp
bool hayAlguno   = lista.Any();                         // ¿tiene elementos?
bool hayCaros    = lista.Any(p => p.Precio > 1000);     // ¿alguno cumple?
bool todosActivos = lista.All(p => p.Activo);           // ¿todos cumplen?
bool tieneId5    = lista.Contains(productoX);           // ¿contiene el objeto?
int  cantidad    = lista.Count();                       // total de elementos
int  cantCaros   = lista.Count(p => p.Precio > 500);    // total que cumplen
long cantLarga   = lista.LongCount();                   // para colecciones grandes

// Contains con comparer
bool existeNombre = nombres.Select(n => n.ToUpper())
    .Contains("LAPTOP HP", StringComparer.OrdinalIgnoreCase);

// TryGetNonEnumeratedCount — evita iterar (.NET 6+)
if (lista.TryGetNonEnumeratedCount(out int count))
    Console.WriteLine($"Total: {count} (sin iterar)");
```

### 4.10 Conversion y Generacion

```csharp
// ToList / ToArray — materializar
List<Producto> listaM = productos.Where(p => p.Activo).ToList();
Producto[] array = productos.OrderBy(p => p.Precio).ToArray();

// ToDictionary — clave-valor (clave unica)
var dict = productos.ToDictionary(p => p.Id, p => p.Nombre);
var dictCompleto = productos.ToDictionary(p => p.Id);

// ToHashSet — conjunto sin duplicados
var hashSet = productos.Select(p => p.Categoria).ToHashSet();

// AsEnumerable — forzar evaluacion en cliente
var enCliente = ctx.Productos.AsEnumerable().Where(p => MiFiltro(p));

// AsQueryable — mantener evaluacion en servidor
var enServidor = lista.AsQueryable().Where(p => p.Precio > 100);

// Range — generar secuencia de numeros
var numeros = Enumerable.Range(1, 100);  // 1, 2, 3, ..., 100
var anos = Enumerable.Range(2020, 7)     // 2020-2026
    .Select(a => new { Ano = a, EsBisiesto = DateTime.IsLeapYear(a) });

// Repeat — repetir un valor
var ceros = Enumerable.Repeat(0, 10);    // [0,0,0,0,0,0,0,0,0,0]

// Empty — secuencia vacia tipada
var vacio = Enumerable.Empty<Producto>();

// Chunk — dividir en lotes (.NET 6+)
var lotes = Enumerable.Range(1, 100).Chunk(10); // 10 lotes de 10 elementos
```

### Tabla Resumen — Todos los Operadores

| Categoria | Operadores | Ejecucion |
|-----------|-----------|-----------|
| Filtrado | `Where`, `OfType` | Diferida |
| Proyeccion | `Select`, `SelectMany` | Diferida |
| Ordenamiento | `OrderBy`, `OrderByDescending`, `ThenBy`, `ThenByDescending`, `Reverse` | Diferida |
| Agrupacion | `GroupBy`, `ToLookup` | Diferida / Inmediata |
| Join | `Join`, `GroupJoin`, `Zip` | Diferida |
| Agregado | `Count`, `LongCount`, `Sum`, `Min`, `Max`, `Average`, `Aggregate`, `MinBy`, `MaxBy` | Inmediata |
| Elementos | `First`, `FirstOrDefault`, `Last`, `LastOrDefault`, `Single`, `SingleOrDefault`, `ElementAt`, `ElementAtOrDefault` | Inmediata |
| Cuantificacion | `Any`, `All`, `Contains` | Inmediata |
| Particion | `Take`, `Skip`, `TakeWhile`, `SkipWhile` | Diferida |
| Conjuntos | `Distinct`, `DistinctBy`, `Union`, `UnionBy`, `Intersect`, `IntersectBy`, `Except`, `ExceptBy`, `Concat` | Diferida |
| Conversion | `ToArray`, `ToList`, `ToDictionary`, `ToLookup`, `ToHashSet`, `AsEnumerable`, `AsQueryable`, `Cast` | Inmediata |
| Generacion | `Range`, `Repeat`, `Empty`, `Chunk`, `DefaultIfEmpty` | Inmediata |

---

## 5. Funciones de Agregado — Guia Completa

Las funciones de agregado **reducen una coleccion a un unico valor**. Son esenciales para generar reportes, estadisticas y dashboard en cualquier aplicacion empresarial. En la arquitectura de 4 capas, las funciones de agregado se utilizan intensivamente en la **Capa de Logica de Negocio (BLL)** para calcular metricas que luego se presentan al usuario, y en la **Capa de Repositorio (DAL)** para consultas optimizadas que se traducen a SQL.

### 5.1 Count y LongCount — Contar

```csharp
var ventas = new List<Venta> { /* ... */ };

// Contar todos los elementos
int total = ventas.Count();

// Contar con condicion
int ventasGrandes = ventas.Count(v => v.Monto > 10_000);

// LongCount para colecciones muy grandes (> 2 mil millones)
long totalGrande = datosGrandes.LongCount();

// Contar en GroupBy — ventas por vendedor
var ventasPorVendedor = ventas
    .GroupBy(v => v.VendedorId)
    .Select(g => new {
        VendedorId = g.Key,
        TotalVentas = g.Count(),
        VentasGrandes = g.Count(v => v.Monto > 5000)
    })
    .OrderByDescending(x => x.TotalVentas);

// Count en repositorio (DAL)
public async Task<int> ContarActivosAsync() =>
    await _ctx.Productos.CountAsync(p => p.Activo);
```

### 5.2 Sum — Sumar

```csharp
// Suma simple
decimal totalIngresos = ventas.Sum(v => v.Monto);

// Suma condicional — ingresos del mes actual
decimal ingresosMes = ventas
    .Where(v => v.Fecha.Month == DateTime.Now.Month
             && v.Fecha.Year == DateTime.Now.Year)
    .Sum(v => v.Monto);

// Suma agrupada por mes
var ingresosPorMes = ventas
    .GroupBy(v => new { v.Fecha.Year, v.Fecha.Month })
    .Select(g => new {
        Ano = g.Key.Year,
        Mes = g.Key.Month,
        Total = g.Sum(v => v.Monto),
        Cantidad = g.Count(),
        Promedio = g.Average(v => v.Monto)
    })
    .OrderBy(x => x.Ano).ThenBy(x => x.Mes);

// Suma con calculo compuesto
decimal valorInventario = productos.Sum(p => p.Precio * p.Stock);

// Suma en EF Core (se traduce a SUM en SQL)
var totalBD = await _ctx.Ventas
    .Where(v => v.Fecha.Year == 2024)
    .SumAsync(v => v.Monto);

// Suma por categoria — con GroupBy
var inventarioPorCategoria = productos
    .GroupBy(p => p.Categoria)
    .Select(g => new {
        Categoria = g.Key,
        ValorTotal = g.Sum(p => p.Precio * p.Stock),
        StockTotal = g.Sum(p => p.Stock),
        Cantidad = g.Count()
    })
    .OrderByDescending(x => x.ValorTotal);
```

### 5.3 Average — Promedio

```csharp
// Promedio simple
double promedioPrecio = productos.Average(p => (double)p.Precio);

// Promedio por categoria
var promedioPorCategoria = productos
    .GroupBy(p => p.Categoria)
    .Select(g => new {
        Categoria = g.Key,
        Promedio = g.Average(p => p.Precio),
        CantidadProductos = g.Count(),
        Mediana = g.OrderBy(p => p.Precio)
                   .Skip(g.Count() / 2)
                   .FirstOrDefault().Precio
    });

// Promedio ponderado
var promedioPonderado = ventas
    .GroupBy(v => v.ProductoId)
    .Select(g => new {
        ProductoId = g.Key,
        PromedioPonderado = g.Sum(v => v.Monto * v.Cantidad)
                          / g.Sum(v => v.Cantidad)
    });

// Promedio en EF Core
var avgBD = await _ctx.Productos
    .Where(p => p.Activo)
    .AverageAsync(p => p.Precio);
```

### 5.4 Min y Max — Minimo y Maximo

```csharp
// Valores extremos simples
decimal precioMin = productos.Min(p => p.Precio);
decimal precioMax = productos.Max(p => p.Precio);

// Obtener el OBJETO con el precio minimo/maximo (.NET 6+)
var masBarato  = productos.MinBy(p => p.Precio);
var masCaro    = productos.MaxBy(p => p.Precio);

// Compatibilidad .NET 5 y anteriores
var masCaro5 = productos
    .OrderByDescending(p => p.Precio)
    .FirstOrDefault();

// Min/Max por grupo
var extremosPorCategoria = productos
    .GroupBy(p => p.Categoria)
    .Select(g => new {
        Categoria = g.Key,
        MasBarato = g.Min(p => p.Precio),
        MasCaro   = g.Max(p => p.Precio),
        Rango     = g.Max(p => p.Precio) - g.Min(p => p.Precio),
        ProductoBarato = g.OrderBy(p => p.Precio).First().Nombre,
        ProductoCaro   = g.OrderByDescending(p => p.Precio).First().Nombre
    });

// Min/Max con fechas
var fechaMin = ventas.Min(v => v.Fecha);
var fechaMax = ventas.Max(v => v.Fecha);
var antiguedad = DateTime.Now - fechaMin;

// Min/Max en EF Core
var precioMinBD = await _ctx.Productos.MinAsync(p => p.Precio);
var precioMaxBD = await _ctx.Productos.MaxAsync(p => p.Precio);
```

### 5.5 Aggregate — Reduccion Personalizada

`Aggregate` es el operador de agregado mas poderoso y flexible. Aplica una funcion acumuladora sobre la secuencia y produce un resultado final. Se usa para calculos personalizados que no se pueden expresar con `Sum`, `Min`, `Max` o `Average`.

```csharp
// Ejemplo 1: Productorio (multiplicacion de todos los numeros)
int[] numeros = { 1, 2, 3, 4, 5 };
int productorio = numeros.Aggregate(1, (acum, n) => acum * n);
// Resultado: 120  (1x2x3x4x5)

// Ejemplo 2: Construir una cadena de texto
var palabras = new[] { "LINQ", "es", "poderoso" };
string frase = palabras.Aggregate((actual, siguiente) =>
    actual + " " + siguiente);
// Resultado: "LINQ es poderoso"

// Ejemplo 3: Encontrar el numero mayor manualmente
int maximo = numeros.Aggregate((max, n) => n > max ? n : max);

// Ejemplo 4: Acumular con semilla y proyeccion final
string resumen = ventas.Aggregate(
    seed: 0m,                                         // valor inicial
    func: (total, v) => total + v.Monto,             // acumulacion
    resultSelector: total => $"Total: ${total:N2}"   // proyeccion final
);

// Ejemplo 5: Calcular estadisticas completas en una sola pasada
var stats = productos.Aggregate(
    new { Min = decimal.MaxValue, Max = 0m, Total = 0m, Count = 0 },
    (acc, p) => new {
        Min   = Math.Min(acc.Min, p.Precio),
        Max   = Math.Max(acc.Max, p.Precio),
        Total = acc.Total + p.Precio,
        Count = acc.Count + 1
    },
    result => new {
        result.Min,
        result.Max,
        result.Total,
        Promedio = result.Total / result.Count,
        Rango = result.Max - result.Min
    });

// Ejemplo 6: Desviacion estandar
var desvStd = ventas.Select(v => (double)v.Monto)
    .Aggregate(
        new { Sum = 0.0, SumSq = 0.0, Count = 0 },
        (acc, val) => new {
            Sum   = acc.Sum + val,
            SumSq = acc.SumSq + val * val,
            Count = acc.Count + 1
        },
        r => Math.Sqrt(r.SumSq / r.Count - Math.Pow(r.Sum / r.Count, 2)));

// Ejemplo 7: Diccionario de frecuencia
var frecuencia = productos
    .Select(p => p.Categoria)
    .Aggregate(
        new Dictionary<string, int>(),
        (acc, cat) =>
        {
            if (acc.ContainsKey(cat)) acc[cat]++;
            else acc[cat] = 1;
            return acc;
        });

// Ejemplo 8: StringBuilder para concatenacion eficiente
var sb = productos.Aggregate(
    new StringBuilder(),
    (builder, p) => builder.AppendLine($"{p.Nombre}: ${p.Precio:F2}"),
    builder => builder.ToString());
```

### 5.6 Dashboard de Agregados — Multiples Metricas

```csharp
public class ResumenVentas
{
    public int     TotalVentas    { get; set; }
    public decimal MontoTotal     { get; set; }
    public decimal PromedioVenta  { get; set; }
    public decimal VentaMinima    { get; set; }
    public decimal VentaMaxima    { get; set; }
    public decimal DesviacionStd  { get; set; }
    public decimal Mediana        { get; set; }
}

// Metodo 1: Con GroupBy (una sola pasada por la coleccion)
var resumen = ventas
    .GroupBy(_ => 1)           // agrupa todo en un solo grupo
    .Select(g => new ResumenVentas
    {
        TotalVentas    = g.Count(),
        MontoTotal     = g.Sum(v => v.Monto),
        PromedioVenta  = g.Average(v => v.Monto),
        VentaMinima    = g.Min(v => v.Monto),
        VentaMaxima    = g.Max(v => v.Monto),
        Mediana = g.OrderBy(v => v.Monto)
                   .Skip(g.Count() / 2)
                   .FirstOrDefault().Monto
    })
    .FirstOrDefault();

// Metodo 2: Con Aggregate (una sola iteracion, mas eficiente)
var stats = ventas.Aggregate(
    new {
        Count = 0, Sum = 0m, Min = decimal.MaxValue,
        Max = decimal.MinValue, SumSq = 0.0
    },
    (acc, v) => new {
        Count = acc.Count + 1,
        Sum   = acc.Sum + v.Monto,
        Min   = Math.Min(acc.Min, v.Monto),
        Max   = Math.Max(acc.Max, v.Monto),
        SumSq = acc.SumSq + (double)v.Monto * (double)v.Monto
    },
    r => new ResumenVentas {
        TotalVentas   = r.Count,
        MontoTotal    = r.Sum,
        PromedioVenta = r.Sum / r.Count,
        VentaMinima   = r.Min,
        VentaMaxima   = r.Max,
        DesviacionStd = (decimal)Math.Sqrt(r.SumSq / r.Count - Math.Pow(r.Sum / r.Count, 2))
    });

// Metodo 3: Con EF Core (se traduce a una sola consulta SQL)
var resumenBD = await _ctx.Ventas
    .Where(v => v.Fecha.Year == 2024)
    .GroupBy(_ => 1)
    .Select(g => new ResumenVentas
    {
        TotalVentas   = g.Count(),
        MontoTotal    = g.Sum(v => v.Monto),
        PromedioVenta = g.Average(v => v.Monto),
        VentaMinima   = g.Min(v => v.Monto),
        VentaMaxima   = g.Max(v => v.Monto)
    })
    .FirstOrDefaultAsync();
```

### 5.7 Agregados con GroupBy — Reportes Comunes

```csharp
// Reporte: Resumen completo por categoria
var reporteCategoria = productos
    .GroupBy(p => p.Categoria)
    .Select(g => new {
        Categoria      = g.Key,
        Cantidad       = g.Count(),
        PrecioMin      = g.Min(p => p.Precio),
        PrecioMax      = g.Max(p => p.Precio),
        PrecioPromedio = g.Average(p => p.Precio),
        StockTotal     = g.Sum(p => p.Stock),
        ValorInventario = g.Sum(p => p.Precio * p.Stock),
        Activos        = g.Count(p => p.Activo),
        Agotados       = g.Count(p => p.Stock == 0)
    })
    .OrderByDescending(x => x.ValorInventario);

// Reporte: Ranking de vendedores
var rankingVendedores = ventas
    .GroupBy(v => v.VendedorId)
    .Select(g => new {
        VendedorId     = g.Key,
        TotalVentas    = g.Count(),
        IngresoTotal   = g.Sum(v => v.Monto),
        TicketPromedio = g.Average(v => v.Monto),
        VentaMaxima    = g.Max(v => v.Monto),
        ProductosDistintos = g.Select(v => v.ProductoId).Distinct().Count()
    })
    .OrderByDescending(x => x.IngresoTotal);

// Reporte: Tendencia mensual con comparativa
var tendenciaMensual = ventas
    .GroupBy(v => new { v.Fecha.Year, v.Fecha.Month })
    .Select(g => new {
        Ano = g.Key.Year,
        Mes = g.Key.Month,
        Total = g.Sum(v => v.Monto),
        Cantidad = g.Count()
    })
    .OrderBy(x => x.Ano).ThenBy(x => x.Mes)
    .ToList();

// Agregar variacion porcentual mes a mes
var conVariacion = tendenciaMensual
    .Select((item, index) => new {
        item.Ano,
        item.Mes,
        item.Total,
        item.Cantidad,
        Variacion = index > 0
            ? Math.Round((item.Total - tendenciaMensual[index - 1].Total)
                       / tendenciaMensual[index - 1].Total * 100, 2)
            : 0
    });
```

### 5.8 Tabla Comparativa — Agregados con y sin LINQ

| Operacion | Sin LINQ (imperativo) | Con LINQ (declarativo) |
|-----------|----------------------|----------------------|
| Contar activos | `int c=0; foreach(var p in list) if(p.Activo) c++;` | `list.Count(p => p.Activo)` |
| Sumar precios | `decimal s=0; foreach(var p in list) s+=p.Precio;` | `list.Sum(p => p.Precio)` |
| Encontrar maximo | `decimal m=0; foreach(var p in list) if(p.Precio>m) m=p.Precio;` | `list.Max(p => p.Precio)` |
| Encontrar objeto max | Ordenar y tomar primero | `list.MaxBy(p => p.Precio)` |
| Promedio por grupo | Dictionary + loop + division | `GroupBy().Select(g => g.Average())` |
| Desviacion estandar | Dos bucles + Math.Sqrt | `Aggregate con SumSq` |

---

## 6. LINQ con Colecciones

### Modelos de Ejemplo

```csharp
// Modelos base usados en todos los ejemplos
public class Producto
{
    public int     Id        { get; set; }
    public string  Nombre    { get; set; } = "";
    public string  Categoria { get; set; } = "";
    public decimal Precio    { get; set; }
    public int     Stock     { get; set; }
    public bool    Activo    { get; set; }
    public List<string> Etiquetas { get; set; } = new();

    // Propiedad calculada
    public decimal PrecioConIVA => Precio * 1.16m;
    public string  Estado => Stock > 20 ? "Disponible" :
                            Stock > 0  ? "Bajo Stock" : "Agotado";
}

public class Cliente
{
    public int    Id     { get; set; }
    public string Nombre { get; set; } = "";
    public string Email  { get; set; } = "";
    public string Ciudad { get; set; } = "";
    public List<Pedido> Pedidos { get; set; } = new();
}

public class Venta
{
    public int      Id         { get; set; }
    public int      ClienteId  { get; set; }
    public int      ProductoId { get; set; }
    public decimal  Monto      { get; set; }
    public int      Cantidad   { get; set; }
    public DateTime Fecha      { get; set; }
    public string   VendedorId { get; set; } = "";
}

// Datos de ejemplo
var productos = new List<Producto>
{
    new() { Id = 1, Nombre = "Laptop HP",         Categoria = "Computo",        Precio = 1200m, Stock = 15,  Activo = true,  Etiquetas = {"computo","premium"} },
    new() { Id = 2, Nombre = "Mouse Logitech",    Categoria = "Accesorio",      Precio = 35m,   Stock = 100, Activo = true,  Etiquetas = {"accesorio","economico"} },
    new() { Id = 3, Nombre = "Teclado Mecanico",  Categoria = "Accesorio",      Precio = 80m,   Stock = 45,  Activo = true,  Etiquetas = {"accesorio","gaming"} },
    new() { Id = 4, Nombre = "Monitor Samsung",   Categoria = "Computo",        Precio = 450m,  Stock = 8,   Activo = true,  Etiquetas = {"computo","display"} },
    new() { Id = 5, Nombre = "SSD Kingston 480GB",Categoria = "Almacenamiento", Precio = 55m,   Stock = 60,  Activo = true,  Etiquetas = {"almacenamiento","ssd"} },
    new() { Id = 6, Nombre = "RAM DDR4 16GB",     Categoria = "Almacenamiento", Precio = 75m,   Stock = 50,  Activo = true,  Etiquetas = {"almacenamiento","ram"} },
    new() { Id = 7, Nombre = "Impresora Epson",   Categoria = "Impresion",      Precio = 280m,  Stock = 12,  Activo = true,  Etiquetas = {"impresion"} },
    new() { Id = 8, Nombre = "Auriculares Sony",  Categoria = "Accesorio",      Precio = 120m,  Stock = 25,  Activo = true,  Etiquetas = {"accesorio","audio"} },
};
```

### Consultas con Tipos Anonimos

```csharp
// Proyectar solo los campos necesarios
var catalogo = productos
    .Where(p => p.Activo && p.Stock > 0)
    .Select(p => new {
        p.Id,
        p.Nombre,
        PrecioFormateado = $"${p.Precio:N2}",
        ConIVA = $"${p.PrecioConIVA:N2}",
        Disponible = p.Estado
    })
    .ToList();

// Salida:
// Id=1, Nombre="Laptop HP", PrecioFormateado="$1,200.00", ConIVA="$1,392.00", Disponible="Disponible"
```

### Consultas Anidadas (Sub-queries)

```csharp
// IDs de productos con stock bajo
var idsStockBajo = productos
    .Where(p => p.Stock < 10)
    .Select(p => p.Id)
    .ToHashSet();

// Ventas de esos productos
var ventasRiesgo = ventas
    .Where(v => idsStockBajo.Contains(v.ProductoId))
    .ToList();

// Productos cuyo precio esta por encima del promedio de su categoria
var sobrePromedio = productos
    .GroupBy(p => p.Categoria)
    .SelectMany(g => g.Where(p => p.Precio > g.Average(x => x.Precio)));

// Top 3 productos mas caros de cada categoria
var topPorCategoria = productos
    .GroupBy(p => p.Categoria)
    .Select(g => new {
        Categoria = g.Key,
        Top = g.OrderByDescending(p => p.Precio).Take(3)
    });
```

### Diccionarios desde LINQ

```csharp
// ToLookup — como Dictionary pero admite claves duplicadas
var ventasPorCliente = ventas.ToLookup(v => v.ClienteId);
var ventasCliente5 = ventasPorCliente[5]; // O(1) acceso

// ToDictionary — una entrada por clave (lanza si hay duplicados)
var productoPorId = productos.ToDictionary(p => p.Id);
var productoX = productoPorId[42]; // O(1) lookup

// ToDictionary con selector de valor
var nombrePorId = productos.ToDictionary(p => p.Id, p => p.Nombre);

// ToDictionary con comparer personalizado
var dictCaseInsensitive = productos.ToDictionary(
    p => p.Nombre,
    p => p,
    StringComparer.OrdinalIgnoreCase);
```

### LINQ con Strings

```csharp
string texto = "LINQ es poderoso y flexible para consultar datos";

// Palabras con mas de 3 letras ordenadas por longitud
var palabras = texto.Split(' ')
    .Where(p => p.Length > 3)
    .OrderBy(p => p.Length)
    .Select(p => $"{p} ({p.Length})");

// Contar frecuencia de caracteres
var frecuencia = texto.ToLower()
    .Where(c => char.IsLetter(c))
    .GroupBy(c => c)
    .Select(g => new { Caracter = g.Key, Frecuencia = g.Count() })
    .OrderByDescending(x => x.Frecuencia);

// Verificar si es pangrama
bool esPangrama = "abcdefghijklmnopqrstuvwxyz"
    .All(c => texto.ToLower().Contains(c));
```

---

## 7. LINQ con Entity Framework

Entity Framework Core convierte LINQ en SQL automaticamente. Es el proveedor LINQ mas utilizado para bases de datos y reside exclusivamente en la Capa de Acceso a Datos (DAL).

### Configuracion Basica

```csharp
// DbContext
public class TiendaContext : DbContext
{
    public TiendaContext(DbContextOptions<TiendaContext> options) : base(options) { }

    public DbSet<Producto> Productos { get; set; }
    public DbSet<Cliente>  Clientes  { get; set; }
    public DbSet<Venta>    Ventas    { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Producto>(entity =>
        {
            entity.HasKey(e => e.Id);
            entity.Property(e => e.Precio).HasColumnType("decimal(18,2)");
            entity.HasOne(e => e.Categoria)
                  .WithMany(c => c.Productos)
                  .HasForeignKey(e => e.CategoriaId)
                  .OnDelete(DeleteBehavior.Restrict);
        });

        // Seed data
        modelBuilder.Entity<Categoria>().HasData(
            new Categoria { Id = 1, Nombre = "Computo" },
            new Categoria { Id = 2, Nombre = "Accesorio" },
            new Categoria { Id = 3, Nombre = "Almacenamiento" },
            new Categoria { Id = 4, Nombre = "Impresion" }
        );
    }
}
```

### Consultas EF Core + LINQ

```csharp
using var ctx = new TiendaContext();

// Se traduce a: SELECT * FROM Productos WHERE Precio > 100 AND Activo = 1
var productosCaros = await ctx.Productos
    .Where(p => p.Precio > 100 && p.Activo)
    .OrderBy(p => p.Nombre)
    .ToListAsync();

// Include — carga relaciones (JOIN en SQL)
var ventasConCliente = await ctx.Ventas
    .Include(v => v.Cliente)
    .Include(v => v.Producto)
        .ThenInclude(p => p.Categoria)
    .Where(v => v.Fecha.Year == 2024)
    .ToListAsync();

// Proyeccion — evita cargar columnas innecesarias
var resumen = await ctx.Ventas
    .Where(v => v.Fecha >= DateTime.Now.AddMonths(-1))
    .GroupBy(v => v.ClienteId)
    .Select(g => new {
        ClienteId  = g.Key,
        TotalGasto = g.Sum(v => v.Monto),
        NumPedidos = g.Count()
    })
    .OrderByDescending(x => x.TotalGasto)
    .Take(10)
    .ToListAsync();

// AsNoTracking — consultas de solo lectura (mejor rendimiento)
var productos = await ctx.Productos
    .AsNoTracking()
    .Where(p => p.Activo)
    .OrderBy(p => p.Nombre)
    .ToListAsync();

// AsSplitQuery — evitar problema Cartesian Explosion (.NET 6+)
var resultado = await ctx.Productos
    .Include(p => p.Categoria)
    .Include(p => p.Ventas)
    .AsSplitQuery()     // Genera multiples queries SQL en vez de un solo JOIN
    .ToListAsync();
```

### Consultas Raw SQL con LINQ

```csharp
// FromSqlRaw — consulta SQL pura
var productos = ctx.Productos
    .FromSqlRaw("SELECT * FROM Productos WHERE Precio > {0}", 100)
    .Where(p => p.Activo)  // LINQ se combina con SQL
    .OrderBy(p => p.Precio)
    .ToList();

// FromSqlInterpolated — SQL con interpolacion segura
var minimo = 100m;
var resultado = ctx.Productos
    .FromSqlInterpolated($"SELECT * FROM Productos WHERE Precio > {minimo}")
    .Include(p => p.Categoria)
    .ToList();

// ExecuteSqlRaw — para INSERT, UPDATE, DELETE directos
int filas = ctx.Database.ExecuteSqlRaw(
    "UPDATE Productos SET Stock = Stock + {0} WHERE CategoriaId = {1}", 10, 1);
```

### Stored Procedures con LINQ

```csharp
// SP que retorna entidades
var productos = ctx.Productos
    .FromSqlRaw("EXEC sp_ObtenerProductosPorCategoria @p0", categoriaId)
    .ToList();

// SP que retorna valores escalares
var total = ctx.Database
    .SqlQueryRaw<decimal>("EXEC sp_CalcularTotalVentas @p0, @p1", fechaInicio, fechaFin)
    .First();

// SP con parametros de salida
var paramOut = new[]
{
    new SqlParameter("@CategoriaId", categoriaId),
    new SqlParameter("@Total", SqlDbType.Decimal) { Direction = ParameterDirection.Output }
};
ctx.Database.ExecuteSqlRaw(
    "EXEC @Total = sp_CalcularTotalPorCategoria @CategoriaId, @Total OUTPUT", paramOut);
decimal resultado = (decimal)paramOut[1].Value;
```

### Migraciones EF Core

```bash
# Instalar herramientas EF Core
dotnet tool install --global dotnet-ef

# Crear la primera migracion
dotnet ef migrations add InitialCreate --project DAL --startup-project Presentacion

# Aplicar migracion a SQL Server
dotnet ef database update --project DAL --startup-project Presentacion

# Crear migracion posterior
dotnet ef migrations add AgregarCampoDescripcion --project DAL --startup-project Presentacion

# Eliminar ultima migracion (antes de aplicar)
dotnet ef migrations remove --project DAL --startup-project Presentacion

# Ver SQL generado por la migracion
dotnet ef migrations script --project DAL --startup-project Presentacion
```

---

## 8. LINQ Async — Patrones Asincronos

Las operaciones asincronas son criticas en aplicaciones web y de servidor para no bloquear hilos mientras se espera la respuesta de la base de datos. EF Core proporciona versiones asincronas de todos los operadores de agregado y materializacion.

### Operaciones Async de Agregado

```csharp
// ToListAsync — materializar async
var productos = await ctx.Productos
    .Where(p => p.Activo)
    .OrderBy(p => p.Nombre)
    .ToListAsync();

// CountAsync — contar async
int total = await ctx.Productos.CountAsync(p => p.Activo);

// SumAsync — sumar async
decimal totalIngresos = await ctx.Ventas.SumAsync(v => v.Monto);

// AverageAsync — promedio async
double promedio = await ctx.Productos.AverageAsync(p => (double)p.Precio);

// MinAsync / MaxAsync — extremos async
decimal minimo = await ctx.Productos.MinAsync(p => p.Precio);
decimal maximo = await ctx.Productos.MaxAsync(p => p.Precio);

// AnyAsync — verificar existencia async
bool existe = await ctx.Productos.AnyAsync(p => p.Nombre == "Laptop HP");

// FirstOrDefaultAsync — primer elemento async
var producto = await ctx.Productos.FirstOrDefaultAsync(p => p.Id == 42);

// SingleOrDefaultAsync — elemento unico async
var admin = await ctx.Usuarios.SingleOrDefaultAsync(u => u.Rol == "Admin");
```

### IAsyncEnumerable — Streaming de Datos (.NET 5+)

```csharp
// StreamAsync — procesar resultados uno por uno sin cargar todos en memoria
await foreach (var producto in ctx.Productos
    .Where(p => p.Activo)
    .AsAsyncEnumerable())
{
    ProcesarProducto(producto); // Se procesa uno a la vez
}

// Metodo que retorna IAsyncEnumerable
public async IAsyncEnumerable<Producto> ObtenerProductosStreamAsync(
    [EnumeratorCancellation] CancellationToken ct = default)
{
    await foreach (var producto in ctx.Productos
        .Where(p => p.Activo)
        .AsAsyncEnumerable()
        .WithCancellation(ct))
    {
        yield return producto;
    }
}
```

### Cancellation Tokens

```csharp
public class ProductoService
{
    private readonly TiendaContext _ctx;

    public async Task<List<Producto>> BuscarAsync(
        string termino, CancellationToken ct = default)
    {
        // El token de cancelacion se propaga a la consulta SQL
        return await _ctx.Productos
            .Where(p => p.Nombre.Contains(termino))
            .OrderBy(p => p.Nombre)
            .ToListAsync(ct);  // Se cancela si ct se activa
    }
}
```

---

## 9. LINQ to XML y JSON

### LINQ to XML — Operaciones Completas

```csharp
using System.Xml.Linq;

// Crear documento XML con LINQ
var doc = new XDocument(
    new XDeclaration("1.0", "utf-8", "yes"),
    new XElement("Inventario",
        new XAttribute("Fecha", DateTime.Now.ToString("yyyy-MM-dd")),
        from p in productos
        select new XElement("Producto",
            new XAttribute("Id", p.Id),
            new XElement("Nombre", p.Nombre),
            new XElement("Precio", p.Precio),
            new XElement("Categoria", p.Categoria),
            new XElement("Stock", p.Stock)
        )
    )
);

// Consultar XML
var caros = from p in doc.Descendants("Producto")
            where (decimal)p.Element("Precio") > 100
            orderby (decimal)p.Element("Precio") descending
            select new {
                Id = (int)p.Attribute("Id"),
                Nombre = (string)p.Element("Nombre"),
                Precio = (decimal)p.Element("Precio")
            };

// Modificar XML — aplicar descuento a una categoria
doc.Descendants("Producto")
    .Where(p => (string)p.Element("Categoria") == "Computo")
    .ToList()
    .ForEach(p => p.Element("Precio")?.SetValue(
        (decimal)p.Element("Precio") * 0.9m));

// Agregar nuevo elemento
doc.Root!.Add(
    new XElement("Producto",
        new XAttribute("Id", 9),
        new XElement("Nombre", "Webcam HD"),
        new XElement("Precio", 65.00m),
        new XElement("Categoria", "Accesorio"),
        new XElement("Stock", 30)
    ));

// Eliminar elementos
doc.Descendants("Producto")
    .Where(p => (int)p.Attribute("Id") == 5)
    .Remove();

// Guardar a archivo
doc.Save("inventario.xml");

// Cargar desde archivo
var docCargado = XDocument.Load("inventario.xml");
```

### LINQ to JSON — System.Text.Json

```csharp
using System.Text.Json;
using System.Text.Json.Nodes;

// Parsear y consultar JSON
string json = """
{
    "productos": [
        {"nombre": "Laptop HP", "precio": 1200, "categoria": "Computo"},
        {"nombre": "Mouse Logitech", "precio": 35, "categoria": "Accesorio"},
        {"nombre": "Monitor Samsung", "precio": 450, "categoria": "Computo"}
    ]
}
""";

var nodo = JsonNode.Parse(json);
var productosJSON = nodo!["productos"]!.AsArray();

// Consultar con LINQ
var caros = productosJSON
    .Where(p => (decimal)p!["precio"]! > 100)
    .Select(p => new {
        Nombre = (string)p!["nombre"]!,
        Precio = (decimal)p!["precio"]!
    });

// Agregados sobre JSON
var estadisticas = productosJSON
    .GroupBy(p => (string)p!["categoria"]!)
    .Select(g => new {
        Categoria = g.Key,
        Cantidad = g.Count(),
        PrecioPromedio = g.Average(p => (decimal)p!["precio"]!)
    });

// Crear JSON con LINQ
var jsonResult = new JsonObject {
    ["resumen"] = new JsonObject {
        ["total"] = productos.Count(),
        ["valorInventario"] = productos.Sum(p => p.Precio * p.Stock),
        ["categorias"] = new JsonArray(
            productos
                .GroupBy(p => p.Categoria)
                .Select(g => (JsonNode)new JsonObject {
                    ["nombre"] = g.Key,
                    ["cantidad"] = g.Count(),
                    ["promedio"] = g.Average(p => (double)p.Precio)
                })
                .ToArray()
        )
    }
};

// Serializar con formato indentado
string jsonFormateado = jsonResult.ToJsonString(
    new JsonSerializerOptions { WriteIndented = true });
Console.WriteLine(jsonFormateado);
```

---

## 10. Expresiones Lambda y Arboles de Expresion

### Func y Action

```csharp
// Func<T, TResult> — delegado generico con retorno
Func<Producto, bool> esCaro = p => p.Precio > 100;
var caros = productos.Where(esCaro);

// Func con multiples parametros
Func<decimal, decimal, decimal> calcularIVA = (precio, tasa) =>
    precio * (1 + tasa);
var conIVA = productos.Select(p => calcularIVA(p.Precio, 0.16m));

// Action<T> — delegado sin retorno
Action<Producto> imprimir = p =>
    Console.WriteLine($"{p.Nombre}: ${p.Precio}");
productos.ToList().ForEach(imprimir);

// Predicate<T> — delegado que retorna bool
Predicate<Producto> filtro = p => p.Activo && p.Stock > 0;
var activos = productos.FindAll(filtro); // FindAll usa Predicate
```

### Expression<T> — Arboles de Expresion

```csharp
// Expression vs Func
Func<Producto, bool> func = p => p.Precio > 100;              // Codigo ejecutable
Expression<Func<Producto, bool>> expr = p => p.Precio > 100;  // Arbol de datos

// EF Core usa Expression para traducir a SQL
var resultado = ctx.Productos.Where(expr).ToList();
// Se traduce a: WHERE Precio > 100

// Construir expresiones dinamicamente
var param = Expression.Parameter(typeof(Producto), "p");
var prop = Expression.Property(param, "Precio");
var constant = Expression.Constant(100m);
var comparison = Expression.GreaterThan(prop, constant);
var lambda = Expression.Lambda<Func<Producto, bool>>(comparison, param);

var dinamicos = ctx.Productos.Where(lambda).ToList();
```

### PredicateBuilder — Filtros Dinamicos

```csharp
public static class PredicateBuilder
{
    public static Expression<Func<T, bool>> True<T>()  => f => true;
    public static Expression<Func<T, bool>> False<T>() => f => false;

    public static Expression<Func<T, bool>> Or<T>(
        this Expression<Func<T, bool>> expr1,
        Expression<Func<T, bool>> expr2) =>
        Expression.Lambda<Func<T, bool>>(
            Expression.OrElse(expr1.Body, expr2.Body),
            expr1.Parameters);

    public static Expression<Func<T, bool>> And<T>(
        this Expression<Func<T, bool>> expr1,
        Expression<Func<T, bool>> expr2) =>
        Expression.Lambda<Func<T, bool>>(
            Expression.AndAlso(expr1.Body, expr2.Body),
            expr1.Parameters);
}

// Uso: Construir filtro dinamicamente
var filtro = PredicateBuilder.True<Producto>();

if (!string.IsNullOrEmpty(categoria))
    filtro = filtro.And(p => p.Categoria == categoria);
if (precioMin.HasValue)
    filtro = filtro.And(p => p.Precio >= precioMin);
if (soloActivos)
    filtro = filtro.And(p => p.Activo);
if (!string.IsNullOrEmpty(busqueda))
    filtro = filtro.And(p => p.Nombre.Contains(busqueda));

var resultado = ctx.Productos.Where(filtro).ToList();
```

> **Tip:** Usa `Func<T>` para LINQ to Objects (en memoria) y `Expression<Func<T>>` para LINQ to SQL / EF Core (base de datos). EF Core no puede traducir `Func<T>` a SQL; necesita `Expression<T>` para analizar la consulta.

---

## 11. Ejecucion Diferida vs Inmediata

La ejecucion diferida (deferred execution) es uno de los conceptos mas importantes de LINQ. La consulta no se ejecuta cuando se define, sino cuando se itera sobre los resultados.

### Ejecucion Diferida (Deferred)

```csharp
// La consulta NO se ejecuta aqui
var consulta = productos.Where(p => p.Precio > 100);

// Se ejecuta AQUI al iterar
foreach (var p in consulta)
    Console.WriteLine(p.Nombre);

// Si los datos cambian, la consulta refleja los cambios
productos.Add(new Producto { Nombre = "Nuevo", Precio = 200 });
// La proxima iteracion de 'consulta' incluira el nuevo producto
```

### Ejecucion Inmediata (Immediate)

```csharp
// ToList() fuerza ejecucion inmediata
var lista = productos.Where(p => p.Precio > 100).ToList();

// Count() ejecuta inmediatamente
int total = productos.Count(p => p.Activo);

// First() ejecuta inmediatamente
var primero = productos.First(p => p.Precio > 500);

// ToDictionary() ejecuta inmediatamente
var dict = productos.ToDictionary(p => p.Id);

// Todos los agregados ejecutan inmediatamente
var suma = productos.Sum(p => p.Precio);
var max = productos.Max(p => p.Precio);
var any = productos.Any(p => p.Stock == 0);
```

### Clasificacion de Operadores

| Tipo | Operadores | Ejecucion |
|------|-----------|-----------|
| **Diferida (Streaming)** | `Where`, `Select`, `SelectMany`, `OrderBy`, `Take`, `Skip`, `Distinct` | Procesa un elemento a la vez |
| **Diferida (Non-streaming)** | `OrderBy`, `GroupBy`, `Reverse`, `Concat` | Necesita todos los datos antes de emitir |
| **Inmediata** | `ToList`, `ToArray`, `Count`, `Sum`, `First`, `Any`, `ToDictionary` | Ejecuta al momento |

> **Precaucion:** Si iteras multiples veces sobre una consulta diferida, la consulta se ejecuta multiples veces. Usa `ToList()` para materializar si necesitas iterar mas de una vez.

---

## 12. PLINQ — LINQ Paralelo

PLINQ ejecuta consultas en multiples nucleos del procesador simultaneamente. Es ideal para operaciones intensivas de CPU sobre grandes colecciones en memoria. **No es adecuado para operaciones de E/S como consultas a bases de datos.**

```csharp
// AsParallel — ejecutar en paralelo
var resultado = datos.AsParallel()
    .Where(x => x.EsValido())
    .Select(x => x.Procesar())
    .ToList();

// AsOrdered — mantener orden original
var ordenado = datos.AsParallel().AsOrdered()
    .Where(x => x.Valor > 100)
    .Select(x => x.Transformar())
    .ToList();

// WithDegreeOfParallelism — limitar hilos
var limitado = datos.AsParallel()
    .WithDegreeOfParallelism(4)
    .Where(x => x.Valor > 1000)
    .ToList();

// WithCancellation — cancelar operacion paralela
var cts = new CancellationTokenSource();
var cancelable = datos.AsParallel()
    .WithCancellation(cts.Token)
    .Select(x => Procesar(x));

// ForAll — ejecutar sin recolectar resultados
datos.AsParallel()
    .Where(p => p.Stock < 10)
    .ForAll(p => EnviarAlertaStock(p));
```

> **Precaucion:** PLINQ no siempre es mas rapido. Para colecciones pequenas u operaciones simples, el overhead de paralelizacion puede ser mayor que el beneficio. Usa `AsParallel()` solo cuando la operacion por elemento sea costosa.

---

## 13. Arquitectura de 4 Capas con LINQ

La arquitectura de 4 capas separa responsabilidades y centraliza el uso de LINQ en la capa de **Repositorio** y **Servicio**. Cada capa utiliza LINQ de manera diferente y para propositos distintos.

```
+-------------------------------------------+
|       CAPA DE PRESENTACION               |  <- Razor / Blazor / MVC / API
|       (Controllers / Pages)              |     Solo llama servicios
+-------------------+-----------------------+
                    |
+-------------------v-----------------------+
|       CAPA DE SERVICIO (BLL)             |  <- Logica de negocio
|       (Business Logic)                   |     Usa repositorios, aplica reglas
+-------------------+-----------------------+     LINQ: GroupBy, Aggregate, Where
                    |
+-------------------v-----------------------+
|       CAPA DE REPOSITORIO (DAL)          |  <- LINQ vive aqui
|       (Data Access / Repository)         |     Consultas a la base de datos
+-------------------+-----------------------+     LINQ: Query Syntax, Include, Where
                    |
+-------------------v-----------------------+
|       CAPA DE DATOS                      |  <- Entity Framework Core
|       (DbContext / Models)               |     Modelos y configuracion EF
+-------------------------------------------+
                    |
            +-------v-------+
            |   SQL Server   |
            +---------------+
```

### Estructura de la Solucion

```
TiendaLinq.sln
+-- src/
|   +-- Tienda.Api/                  <- Proyecto ASP.NET Core
|   |   +-- Controllers/
|   |   |   +-- ProductosController.cs
|   |   |   +-- VentasController.cs
|   |   +-- Program.cs
|   +-- Tienda.Core/                 <- Logica de negocio
|   |   +-- Models/
|   |   +-- DTOs/
|   |   +-- Interfaces/
|   |   +-- Services/
|   +-- Tienda.Data/                 <- Acceso a datos
|       +-- Context/
|       +-- Repositories/
|       +-- Migrations/
+-- tests/
|   +-- Tienda.Tests.Unit/
|   +-- Tienda.Tests.Integration/
+-- .github/
|   +-- workflows/
|       +-- ci.yml
+-- .gitignore
+-- README.md
```

### Capa 1 — Modelos (Data Layer)

```csharp
// Models/Producto.cs
namespace Tienda.Models;

public class Producto
{
    public int      Id             { get; set; }
    public string   Nombre         { get; set; } = "";
    public string   Categoria      { get; set; } = "";
    public decimal  Precio         { get; set; }
    public int      Stock          { get; set; }
    public bool     Activo         { get; set; }
    public DateTime FechaCreacion  { get; set; } = DateTime.Now;

    // Relaciones
    public ICollection<Venta> Ventas { get; set; } = new List<Venta>();
}

// DTOs/ProductoDTO.cs
public class ProductoDTO
{
    public int     Id               { get; set; }
    public string  Nombre           { get; set; } = "";
    public string  Categoria        { get; set; } = "";
    public decimal Precio           { get; set; }
    public string  PrecioFormateado { get; set; } = "";
    public string  Disponibilidad   { get; set; } = "";
}

public class ResumenProductosDto
{
    public int     TotalProductos  { get; set; }
    public decimal ValorInventario { get; set; }
    public decimal PrecioPromedio  { get; set; }
    public decimal PrecioMinimo    { get; set; }
    public decimal PrecioMaximo    { get; set; }
    public int     TotalUnidades   { get; set; }
}

// Context/TiendaContext.cs
public class TiendaContext : DbContext
{
    public TiendaContext(DbContextOptions<TiendaContext> options) : base(options) { }

    public DbSet<Producto> Productos { get; set; }
    public DbSet<Cliente>  Clientes  { get; set; }
    public DbSet<Venta>    Ventas    { get; set; }
}
```

### Capa 2 — Repositorio (Data Access / LINQ)

```csharp
// Repositories/IProductoRepository.cs
namespace Tienda.Repositories;

public interface IProductoRepository
{
    Task<IEnumerable<Producto>> ObtenerTodosAsync();
    Task<IEnumerable<Producto>> ObtenerActivosAsync();
    Task<Producto?>             ObtenerPorIdAsync(int id);
    Task<IEnumerable<Producto>> BuscarAsync(string termino);
    Task<IEnumerable<Producto>> ObtenerPorCategoriaAsync(string categoria);
    Task<IEnumerable<Producto>> ObtenerStockBajoAsync(int umbral = 5);
    Task<ResumenProductosDto>   ObtenerResumenAsync();
    Task<Producto>              CrearAsync(Producto producto);
    Task<Producto>              ActualizarAsync(Producto producto);
    Task                        EliminarAsync(int id);
}

// Repositories/ProductoRepository.cs
public class ProductoRepository : IProductoRepository
{
    private readonly TiendaContext _ctx;

    public ProductoRepository(TiendaContext ctx) => _ctx = ctx;

    // LINQ: obtener todos los activos ordenados
    public async Task<IEnumerable<Producto>> ObtenerActivosAsync() =>
        await _ctx.Productos
            .Where(p => p.Activo)
            .OrderBy(p => p.Categoria)
            .ThenBy(p => p.Nombre)
            .AsNoTracking()
            .ToListAsync();

    // LINQ: busqueda de texto
    public async Task<IEnumerable<Producto>> BuscarAsync(string termino) =>
        await _ctx.Productos
            .Where(p => p.Activo &&
                        (p.Nombre.Contains(termino) ||
                         p.Categoria.Contains(termino)))
            .OrderBy(p => p.Nombre)
            .AsNoTracking()
            .ToListAsync();

    // LINQ: stock bajo
    public async Task<IEnumerable<Producto>> ObtenerStockBajoAsync(int umbral = 5) =>
        await _ctx.Productos
            .Where(p => p.Activo && p.Stock <= umbral)
            .OrderBy(p => p.Stock)
            .AsNoTracking()
            .ToListAsync();

    // LINQ: resumen con AGREGADOS
    public async Task<ResumenProductosDto> ObtenerResumenAsync()
    {
        var stats = await _ctx.Productos
            .Where(p => p.Activo)
            .GroupBy(_ => 1)
            .Select(g => new ResumenProductosDto
            {
                TotalProductos   = g.Count(),
                ValorInventario  = g.Sum(p => p.Precio * p.Stock),
                PrecioPromedio   = g.Average(p => p.Precio),
                PrecioMinimo     = g.Min(p => p.Precio),
                PrecioMaximo     = g.Max(p => p.Precio),
                TotalUnidades    = g.Sum(p => p.Stock)
            })
            .FirstOrDefaultAsync();

        return stats ?? new ResumenProductosDto();
    }

    public async Task<Producto> CrearAsync(Producto producto)
    {
        _ctx.Productos.Add(producto);
        await _ctx.SaveChangesAsync();
        return producto;
    }

    public async Task<Producto> ActualizarAsync(Producto producto)
    {
        _ctx.Productos.Update(producto);
        await _ctx.SaveChangesAsync();
        return producto;
    }

    public async Task EliminarAsync(int id)
    {
        var producto = await _ctx.Productos.FindAsync(id);
        if (producto != null)
        {
            producto.Activo = false; // Soft delete
            await _ctx.SaveChangesAsync();
        }
    }
}
```

### Capa 3 — Servicio (Business Logic)

```csharp
// Services/IProductoService.cs
namespace Tienda.Services;

public interface IProductoService
{
    Task<IEnumerable<ProductoDTO>> ObtenerCatalogoAsync();
    Task<ProductoDTO?>             ObtenerDetalleAsync(int id);
    Task<IEnumerable<ProductoDTO>> BuscarAsync(string termino);
    Task<ResumenProductosDto>      ObtenerResumenAsync();
    Task<ProductoDTO>              CrearProductoAsync(CrearProductoDto dto);
    Task<bool>                     ActualizarPrecioAsync(int id, decimal nuevoPrecio);
}

// Services/ProductoService.cs
public class ProductoService : IProductoService
{
    private readonly IProductoRepository _repo;
    private readonly ILogger<ProductoService> _logger;

    public ProductoService(IProductoRepository repo, ILogger<ProductoService> logger)
    {
        _repo   = repo;
        _logger = logger;
    }

    public async Task<IEnumerable<ProductoDTO>> ObtenerCatalogoAsync()
    {
        var productos = await _repo.ObtenerActivosAsync();

        // LINQ en la capa de servicio: transformacion a DTO
        return productos.Select(p => new ProductoDTO
        {
            Id               = p.Id,
            Nombre           = p.Nombre,
            Categoria        = p.Categoria,
            Precio           = p.Precio,
            PrecioFormateado = $"${p.Precio:N2}",
            Disponibilidad   = p.Stock switch
            {
                0     => "Sin stock",
                <= 5  => "Poco stock",
                <= 20 => "Stock normal",
                _     => "Stock alto"
            }
        });
    }

    public async Task<ProductoDTO> CrearProductoAsync(CrearProductoDto dto)
    {
        if (dto.Precio <= 0)
            throw new ArgumentException("El precio debe ser mayor a cero.");

        var producto = new Producto
        {
            Nombre    = dto.Nombre.Trim(),
            Categoria = dto.Categoria,
            Precio    = dto.Precio,
            Stock     = dto.Stock,
            Activo    = true
        };

        var creado = await _repo.CrearAsync(producto);
        _logger.LogInformation("Producto creado: {Id} - {Nombre}", creado.Id, creado.Nombre);

        return new ProductoDTO { Id = creado.Id, Nombre = creado.Nombre };
    }
}
```

### Capa 4 — Presentacion (Controller / API)

```csharp
// Controllers/ProductosController.cs
namespace Tienda.Controllers;

[ApiController]
[Route("api/[controller]")]
public class ProductosController : ControllerBase
{
    private readonly IProductoService _service;

    public ProductosController(IProductoService service) => _service = service;

    [HttpGet]
    public async Task<IActionResult> GetCatalogo() =>
        Ok(await _service.ObtenerCatalogoAsync());

    [HttpGet("buscar")]
    public async Task<IActionResult> Buscar([FromQuery] string q)
    {
        if (string.IsNullOrWhiteSpace(q))
            return BadRequest("El termino de busqueda no puede estar vacio.");
        return Ok(await _service.BuscarAsync(q));
    }

    [HttpGet("resumen")]
    public async Task<IActionResult> GetResumen() =>
        Ok(await _service.ObtenerResumenAsync());

    [HttpPost]
    public async Task<IActionResult> Crear([FromBody] CrearProductoDto dto)
    {
        var creado = await _service.CrearProductoAsync(dto);
        return CreatedAtAction(nameof(GetCatalogo), new { id = creado.Id }, creado);
    }
}
```

### Registro de Dependencias (Program.cs)

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddDbContext<TiendaContext>(opt =>
    opt.UseSqlServer(builder.Configuration.GetConnectionString("Default")));

builder.Services.AddScoped<IProductoRepository, ProductoRepository>();
builder.Services.AddScoped<IProductoService,    ProductoService>();

builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();
app.UseSwagger();
app.UseSwaggerUI();
app.MapControllers();
app.Run();
```

### Resumen de LINQ por Capa

| Capa | Operadores LINQ mas usados | Proposito |
|------|---------------------------|-----------|
| **Presentacion** | `Select`, `OrderBy`, `Take`, `Skip`, `Count`, `Sum` | Formateo, paginacion, resumenes visuales |
| **BLL (Servicio)** | `Where`, `GroupBy`, `Aggregate`, `Any`, `All`, `Select` | Validaciones, reglas de negocio, calculos |
| **DAL (Repositorio)** | `Where`, `Include`, `AsNoTracking`, `FirstOrDefault`, `OrderBy` | Consultas SQL, filtrado, carga de relaciones |
| **Datos (Modelos)** | Data Annotations, `Expression<T>` | Mapeo, validacion de modelos |

---

## 14. Proyecto Integrado — CRUD Completo

### Scripts SQL Server — Creacion de Tablas

```sql
CREATE DATABASE Linq4CapasDB;
GO

USE Linq4CapasDB;
GO

CREATE TABLE Categorias (
    Id            INT IDENTITY(1,1) PRIMARY KEY,
    Nombre        NVARCHAR(100) NOT NULL,
    Descripcion   NVARCHAR(500) NULL,
    Activa        BIT NOT NULL DEFAULT 1,
    FechaCreacion DATETIME2 NOT NULL DEFAULT GETDATE()
);

CREATE TABLE Productos (
    Id            INT IDENTITY(1,1) PRIMARY KEY,
    Nombre        NVARCHAR(200) NOT NULL,
    Descripcion   NVARCHAR(1000) NULL,
    Precio        DECIMAL(18,2) NOT NULL,
    Stock         INT NOT NULL DEFAULT 0,
    CategoriaId   INT NOT NULL FOREIGN KEY REFERENCES Categorias(Id),
    Activo        BIT NOT NULL DEFAULT 1,
    FechaCreacion DATETIME2 NOT NULL DEFAULT GETDATE()
);

CREATE TABLE Empleados (
    Id            INT IDENTITY(1,1) PRIMARY KEY,
    Nombre        NVARCHAR(100) NOT NULL,
    Apellido      NVARCHAR(100) NOT NULL,
    Departamento  NVARCHAR(50) NOT NULL,
    Salario       DECIMAL(18,2) NOT NULL,
    FechaIngreso  DATE NOT NULL,
    Activo        BIT NOT NULL DEFAULT 1
);

CREATE TABLE Ventas (
    Id             INT IDENTITY(1,1) PRIMARY KEY,
    ProductoId     INT NOT NULL FOREIGN KEY REFERENCES Productos(Id),
    Cantidad       INT NOT NULL,
    PrecioUnitario DECIMAL(18,2) NOT NULL,
    FechaVenta     DATETIME2 NOT NULL DEFAULT GETDATE(),
    VendedorId     INT NOT NULL FOREIGN KEY REFERENCES Empleados(Id),
    Total          AS (Cantidad * PrecioUnitario) PERSISTED
);

-- Datos de ejemplo (Seeding)
INSERT INTO Categorias (Nombre, Descripcion) VALUES
('Computo', 'Equipos de computo y accesorios'),
('Accesorio', 'Perifericos y accesorios'),
('Almacenamiento', 'Dispositivos de almacenamiento'),
('Impresion', 'Impresoras y suministros');

INSERT INTO Productos (Nombre, Precio, Stock, CategoriaId) VALUES
('Laptop HP', 1200.00, 15, 1),
('Mouse Logitech', 35.00, 100, 2),
('Teclado Mecanico', 80.00, 45, 2),
('Monitor Samsung', 450.00, 8, 1),
('SSD Kingston 480GB', 55.00, 60, 3),
('RAM DDR4 16GB', 75.00, 50, 3),
('Impresora Epson', 280.00, 12, 4),
('Auriculares Sony', 120.00, 25, 2);

INSERT INTO Empleados (Nombre, Apellido, Departamento, Salario, FechaIngreso) VALUES
('Carlos', 'Lopez', 'Finanzas', 4200.00, '2019-03-15'),
('Ana', 'Garcia', 'Ventas', 3500.00, '2020-06-01'),
('Jorge', 'Fernandez', 'TI', 5500.00, '2018-01-20'),
('Maria', 'Rodriguez', 'TI', 4500.00, '2021-09-10'),
('Pedro', 'Martinez', 'Ventas', 3800.00, '2020-11-05');
```

### Configuracion de Inicio Completa

```csharp
// Presentacion/Program.cs
using Microsoft.EntityFrameworkCore;
using DAL;
using DAL.Interfaces;
using BLL;
using BLL.Interfaces;
using Entidades.DTOs;

var services = new ServiceCollection();

// 1. Registrar DbContext
services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(
        "Server=localhost;Database=Linq4CapasDB;" +
        "Trusted_Connection=True;TrustServerCertificate=True;"));

// 2. Registrar Repositorios (DAL)
services.AddScoped(typeof(IRepositorio<>), typeof(Repositorio<>));
services.AddScoped<ProductoRepositorio>();

// 3. Registrar Servicios (BLL)
services.AddScoped<IProductoServicio, ProductoServicio>();
services.AddScoped<ReporteServicio>();

var serviceProvider = services.BuildServiceProvider();

// 4. Crear base de datos y aplicar migraciones
using (var scope = serviceProvider.CreateScope())
{
    var db = scope.ServiceProvider.GetRequiredService<AppDbContext>();
    db.Database.Migrate();
}

// 5. DEMOSTRACION CRUD COMPLETO
var productoServicio = serviceProvider.GetRequiredService<IProductoServicio>();
var reporteServicio = serviceProvider.GetRequiredService<ReporteServicio>();

// CREATE
int nuevoId = productoServicio.CrearProducto(new ProductoDTO
{
    Nombre = "Webcam HD 1080p", Precio = 65.00m, Stock = 30, Categoria = "2"
});
Console.WriteLine($"Producto creado con ID: {nuevoId}");

// READ
var todos = productoServicio.ObtenerTodos();
todos.Take(5).ToList().ForEach(p =>
    Console.WriteLine($"  {p.Nombre,-25} | ${p.Precio,8:F2} | {p.Estado}"));

// UPDATE
var actualizado = productoServicio.ActualizarProducto(new ProductoDTO
{
    Id = nuevoId, Nombre = "Webcam HD 1080p Pro", Precio = 79.99m, Stock = 25, Categoria = "2"
});

// DELETE (soft delete)
var eliminado = productoServicio.EliminarProducto(nuevoId);

// REPORTES CON AGREGADOS
var stats = productoServicio.EstadisticasPorCategoria();
var top = productoServicio.TopProductos(5);
var resumen = reporteServicio.ResumenInventario();
```

---

## 15. Patrones Avanzados y Operadores Custom

### Extension Methods Personalizados

```csharp
public static class LinqExtensions
{
    // Paginacion generica
    public static IEnumerable<T> Paginar<T>(
        this IEnumerable<T> source, int pagina, int tamano) =>
        source.Skip((pagina - 1) * tamano).Take(tamano);

    // Duplicados
    public static IEnumerable<T> Duplicados<T>(
        this IEnumerable<T> source) =>
        source.GroupBy(x => x)
              .Where(g => g.Count() > 1)
              .Select(g => g.Key);

    // Shuffle — mezclar aleatoriamente
    public static IEnumerable<T> Shuffle<T>(this IEnumerable<T> source) =>
        source.OrderBy(_ => Random.Shared.Next());

    // DistinctBy con expresion (para versiones antes de .NET 6)
    public static IEnumerable<T> DistinctBy<T, TKey>(
        this IEnumerable<T> source, Func<T, TKey> keySelector) =>
        source.GroupBy(keySelector).Select(g => g.First());

    // WhereIf — aplicar filtro condicional
    public static IEnumerable<T> WhereIf<T>(
        this IEnumerable<T> source, bool condition, Func<T, bool> predicate) =>
        condition ? source.Where(predicate) : source;

    // Para IQueryable (EF Core)
    public static IQueryable<T> WhereIf<T>(
        this IQueryable<T> source, bool condition,
        Expression<Func<T, bool>> predicate) =>
        condition ? source.Where(predicate) : source;
}

// Uso
var pagina3 = productos.Paginar(3, 10);
var dups = nombres.Duplicados();
var mezclados = productos.Shuffle();
var filtrado = productos.WhereIf(!string.IsNullOrEmpty(busqueda), p => p.Nombre.Contains(busqueda));
```

### Tabla Pivot con LINQ

```csharp
// Pivot: Ventas por categoria y mes
var pivot = ventas
    .GroupBy(v => new { v.Producto.Categoria, v.Fecha.Month })
    .Select(g => new {
        g.Key.Categoria,
        Mes = g.Key.Month,
        Total = g.Sum(v => v.Cantidad * v.PrecioUnitario)
    })
    .GroupBy(x => x.Categoria)
    .Select(g => new {
        Categoria = g.Key,
        Ene = g.FirstOrDefault(x => x.Mes == 1)?.Total ?? 0,
        Feb = g.FirstOrDefault(x => x.Mes == 2)?.Total ?? 0,
        Mar = g.FirstOrDefault(x => x.Mes == 3)?.Total ?? 0,
        Abr = g.FirstOrDefault(x => x.Mes == 4)?.Total ?? 0,
        May = g.FirstOrDefault(x => x.Mes == 5)?.Total ?? 0,
        Jun = g.FirstOrDefault(x => x.Mes == 6)?.Total ?? 0,
        Total = g.Sum(x => x.Total)
    })
    .OrderByDescending(x => x.Total);
```

### Paginacion Generica con Metadatos

```csharp
public class PagedResult<T>
{
    public IEnumerable<T> Items     { get; set; } = Enumerable.Empty<T>();
    public int            Total     { get; set; }
    public int            Pagina    { get; set; }
    public int            TamanoPag { get; set; }
    public int            TotalPags => (int)Math.Ceiling((double)Total / TamanoPag);
    public bool           HayAnterior => Pagina > 1;
    public bool           HaySiguiente => Pagina < TotalPags;
}

public static class PaginationExtensions
{
    public static async Task<PagedResult<T>> ToPagedAsync<T>(
        this IQueryable<T> query, int pagina, int tamanoPag,
        CancellationToken ct = default)
    {
        var total = await query.CountAsync(ct);
        var items = await query
            .Skip((pagina - 1) * tamanoPag)
            .Take(tamanoPag)
            .ToListAsync(ct);

        return new PagedResult<T>
        {
            Items     = items,
            Total     = total,
            Pagina    = pagina,
            TamanoPag = tamanoPag
        };
    }
}

// Uso
var pagina = await ctx.Productos
    .Where(p => p.Activo)
    .OrderBy(p => p.Nombre)
    .ToPagedAsync(pagina: 2, tamanoPag: 10);
```

---

## 16. Casos de Uso Avanzados

### Filtros Dinamicos con Expression

```csharp
public class FiltroProducto
{
    public string?  Categoria   { get; set; }
    public decimal? PrecioMin   { get; set; }
    public decimal? PrecioMax   { get; set; }
    public bool?    Activo      { get; set; }
    public string?  Busqueda    { get; set; }
    public int?     StockMinimo { get; set; }
}

public static IQueryable<Producto> AplicarFiltros(
    this IQueryable<Producto> query, FiltroProducto filtro)
{
    if (!string.IsNullOrWhiteSpace(filtro.Categoria))
        query = query.Where(p => p.Categoria == filtro.Categoria);

    if (filtro.PrecioMin.HasValue)
        query = query.Where(p => p.Precio >= filtro.PrecioMin.Value);

    if (filtro.PrecioMax.HasValue)
        query = query.Where(p => p.Precio <= filtro.PrecioMax.Value);

    if (filtro.Activo.HasValue)
        query = query.Where(p => p.Activo == filtro.Activo.Value);

    if (!string.IsNullOrWhiteSpace(filtro.Busqueda))
        query = query.Where(p => p.Nombre.Contains(filtro.Busqueda));

    if (filtro.StockMinimo.HasValue)
        query = query.Where(p => p.Stock >= filtro.StockMinimo.Value);

    return query;
}

// Uso
var filtro = new FiltroProducto { Categoria = "Electronica", PrecioMax = 500 };
var resultados = await ctx.Productos.AplicarFiltros(filtro).ToListAsync();
```

### Reporte de Ventas Completo con Agregados

```csharp
public class ReporteVentasService
{
    private readonly TiendaContext _ctx;

    public ReporteVentasService(TiendaContext ctx) => _ctx = ctx;

    public async Task<object> GenerarReporteAnualAsync(int ano)
    {
        var ventas = await _ctx.Ventas
            .Include(v => v.Cliente)
            .Include(v => v.Producto)
            .Where(v => v.Fecha.Year == ano)
            .ToListAsync();

        return new
        {
            Ano = ano,

            // Resumen general con agregados
            TotalVentas    = ventas.Count,
            IngresoTotal   = ventas.Sum(v => v.Monto),
            TicketPromedio = ventas.Average(v => v.Monto),
            VentaMaxima    = ventas.Max(v => v.Monto),
            VentaMinima    = ventas.Min(v => v.Monto),

            // Top 5 clientes por gasto
            TopClientes = ventas
                .GroupBy(v => v.Cliente.Nombre)
                .Select(g => new { Cliente = g.Key, Total = g.Sum(v => v.Monto) })
                .OrderByDescending(x => x.Total)
                .Take(5),

            // Ingresos por mes
            IngresosPorMes = ventas
                .GroupBy(v => v.Fecha.Month)
                .Select(g => new { Mes = g.Key, Total = g.Sum(v => v.Monto) })
                .OrderBy(x => x.Mes),

            // Top 3 categorias mas vendidas
            TopCategorias = ventas
                .GroupBy(v => v.Producto.Categoria)
                .Select(g => new {
                    Categoria = g.Key,
                    Cantidad  = g.Sum(v => v.Cantidad),
                    Ingresos  = g.Sum(v => v.Monto)
                })
                .OrderByDescending(x => x.Ingresos)
                .Take(3)
        };
    }
}
```

---

## 17. LINQ en GitHub — Repositorio de Ejemplo

### .gitignore para Proyectos C#

```gitignore
# Build output
bin/
obj/
*.user
*.suo

# Visual Studio
.vs/
*.csproj.user

# Rider
.idea/

# NuGet
packages/
*.nupkg
*.nuspec

# Secretos y configuracion local
appsettings.Development.json
appsettings.Local.json
*.pfx
*.key

# Cobertura de pruebas
coverage/
TestResults/
```

### GitHub Actions — CI/CD Completo

```yaml
# .github/workflows/ci.yml
name: CI Pipeline - Build, Test & Coverage

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  build-and-test:
    name: Build & Test
    runs-on: ubuntu-latest

    steps:
      - name: Checkout codigo
        uses: actions/checkout@v4

      - name: Setup .NET 8
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '8.0.x'

      - name: Restaurar dependencias
        run: dotnet restore

      - name: Compilar solucion
        run: dotnet build --no-restore --configuration Release

      - name: Ejecutar pruebas con cobertura
        run: |
          dotnet test --no-build --configuration Release \
            --collect:"XPlat Code Coverage" \
            --results-directory ./coverage

      - name: Publicar reporte de cobertura
        uses: codecov/codecov-action@v4
        with:
          directory: ./coverage
          fail_ci_if_error: false

      - name: Publicar API
        if: github.ref == 'refs/heads/main'
        run: dotnet publish src/Tienda.Api -c Release -o ./publish

      - name: Upload artifact
        if: github.ref == 'refs/heads/main'
        uses: actions/upload-artifact@v4
        with:
          name: tienda-api
          path: ./publish
```

### Pruebas Unitarias de LINQ con Agregados

```csharp
// tests/Tienda.Tests.Unit/LinqAgregadosTests.cs
using Xunit;
using FluentAssertions;

public class LinqAgregadosTests
{
    private List<Venta> CrearVentasEjemplo() => new()
    {
        new() { Id = 1, Monto = 500,  ClienteId = 1, Fecha = new DateTime(2024, 1, 15) },
        new() { Id = 2, Monto = 1500, ClienteId = 2, Fecha = new DateTime(2024, 1, 20) },
        new() { Id = 3, Monto = 750,  ClienteId = 1, Fecha = new DateTime(2024, 2, 5)  },
        new() { Id = 4, Monto = 2000, ClienteId = 3, Fecha = new DateTime(2024, 2, 10) },
        new() { Id = 5, Monto = 300,  ClienteId = 2, Fecha = new DateTime(2024, 3, 1)  },
    };

    [Fact]
    public void Sum_DebeCalcularTotalCorrectamente()
    {
        var ventas = CrearVentasEjemplo();
        var total  = ventas.Sum(v => v.Monto);
        total.Should().Be(5050m);
    }

    [Fact]
    public void Average_DebeCalcularPromedioCorrectamente()
    {
        var ventas   = CrearVentasEjemplo();
        var promedio = ventas.Average(v => v.Monto);
        promedio.Should().Be(1010m);
    }

    [Fact]
    public void GroupBy_DebeAgruparPorClienteCorrectamente()
    {
        var ventas = CrearVentasEjemplo();

        var porCliente = ventas
            .GroupBy(v => v.ClienteId)
            .Select(g => new { ClienteId = g.Key, Total = g.Sum(v => v.Monto) })
            .OrderBy(x => x.ClienteId)
            .ToList();

        porCliente.Should().HaveCount(3);
        porCliente[0].Total.Should().Be(1250m);  // Cliente 1: 500 + 750
        porCliente[1].Total.Should().Be(1800m);  // Cliente 2: 1500 + 300
        porCliente[2].Total.Should().Be(2000m);  // Cliente 3: 2000
    }

    [Fact]
    public void Aggregate_DebeReducirCorrectamente()
    {
        var numeros     = new[] { 1, 2, 3, 4, 5 };
        var productorio = numeros.Aggregate(1, (acc, n) => acc * n);
        productorio.Should().Be(120);
    }

    [Fact]
    public void MinBy_MaxBy_DebenEncontrarElementosExtremos()
    {
        var ventas   = CrearVentasEjemplo();
        var masAlta  = ventas.MaxBy(v => v.Monto);
        var masChica = ventas.MinBy(v => v.Monto);

        masAlta!.Monto.Should().Be(2000m);
        masChica!.Monto.Should().Be(300m);
    }

    [Fact]
    public void Count_ConCondicion_DebeContarCorrectamente()
    {
        var ventas = CrearVentasEjemplo();
        var grandes = ventas.Count(v => v.Monto > 1000);
        grandes.Should().Be(2);
    }

    [Fact]
    public void GroupBy_ConAgregados_DebeCalcularEstadisticas()
    {
        var ventas = CrearVentasEjemplo();

        var stats = ventas
            .GroupBy(_ => 1)
            .Select(g => new {
                Count = g.Count(),
                Sum   = g.Sum(v => v.Monto),
                Min   = g.Min(v => v.Monto),
                Max   = g.Max(v => v.Monto)
            })
            .First();

        stats.Count.Should().Be(5);
        stats.Sum.Should().Be(5050m);
        stats.Min.Should().Be(300m);
        stats.Max.Should().Be(2000m);
    }
}
```

---

## 18. Rendimiento y Optimizacion

### Ver el SQL Generado por EF Core

```csharp
// Metodo 1: Logging
var options = new DbContextOptionsBuilder<TiendaContext>()
    .UseSqlServer(connectionString)
    .LogTo(Console.WriteLine, LogLevel.Information)
    .EnableSensitiveDataLogging()  // Muestra valores de parametros
    .Options;

// Metodo 2: ToQueryString() (.NET 6+)
var sql = ctx.Productos
    .Where(p => p.Precio > 100 && p.Activo)
    .OrderBy(p => p.Nombre)
    .ToQueryString();
Console.WriteLine(sql);
// SELECT [p].[Id], [p].[Nombre], [p].[Precio] ...
// FROM [Productos] AS [p]
// WHERE [p].[Precio] > 100.0 AND [p].[Activo] = CAST(1 AS bit)
// ORDER BY [p].[Nombre]
```

### Optimizaciones Clave

```csharp
// 1. AsNoTracking — consultas de solo lectura (hasta 30% mas rapido)
var productos = await ctx.Productos
    .AsNoTracking()
    .Where(p => p.Activo)
    .ToListAsync();

// 2. Proyeccion — solo campos necesarios
var nombres = await ctx.Productos
    .Where(p => p.Activo)
    .Select(p => new { p.Id, p.Nombre })
    .ToListAsync();

// 3. AnyAsync en vez de CountAsync para verificar existencia
bool existe = await ctx.Productos.AnyAsync(p => p.Nombre == "X");
// NO: (await ctx.Productos.CountAsync(p => p.Nombre == "X")) > 0

// 4. AsSplitQuery — evitar Cartesian Explosion (.NET 6+)
var resultado = await ctx.Productos
    .Include(p => p.Categoria)
    .Include(p => p.Ventas)
    .AsSplitQuery()
    .ToListAsync();

// 5. Select para calcular en BD en vez de memoria
// MAL: carga todo y calcula en C#
var productos = await ctx.Productos.ToListAsync();
var total = productos.Sum(p => p.Precio * p.Stock);

// BIEN: calcula en SQL
var total = await ctx.Productos.SumAsync(p => p.Precio * p.Stock);

// 6. Indices en SQL Server para columnas filtradas
// CREATE INDEX IX_Productos_Activo ON Productos(Activo) WHERE Activo = 1;
```

### Comparativa de Rendimiento

| Operacion | En Memoria | En BD (EF Core) | Diferencia |
|-----------|-----------|----------------|------------|
| Filtrar 1000 de 1M | Carga 1M, filtra | WHERE en SQL | 1000x mas rapido |
| Sumar columna | Carga todo, Sum() | SUM() en SQL | Memoria: O(n), SQL: O(1) |
| Contar con filtro | Count() en C# | COUNT() en SQL | SQL usa indices |
| Proyeccion 2 campos | Carga 20 campos | SELECT 2 campos | 10x menos datos transferidos |

---

## 19. Errores Comunes y Buenas Practicas

### Error 1: N+1 Query Problem

```csharp
// MAL — ejecuta 1 SELECT para clientes + N SELECTs para sus ventas
var clientes = await ctx.Clientes.ToListAsync();
foreach (var c in clientes)
{
    var ventas = await ctx.Ventas.Where(v => v.ClienteId == c.Id).ToListAsync();
}

// BIEN — una sola consulta con JOIN
var clientes = await ctx.Clientes
    .Include(c => c.Ventas)
    .ToListAsync();
```

### Error 2: Ejecutar LINQ en Memoria Innecesariamente

```csharp
// MAL — carga TODA la tabla en memoria y filtra en C#
var productos = await ctx.Productos.ToListAsync();
var caros = productos.Where(p => p.Precio > 1000).ToList();

// BIEN — el WHERE se traduce a SQL y filtra en la base de datos
var caros = await ctx.Productos
    .Where(p => p.Precio > 1000)
    .ToListAsync();
```

### Error 3: Olvidar await en Consultas Async

```csharp
// MAL — retorna Task<List<>>, no List<>
var productos = ctx.Productos.ToListAsync();

// BIEN
var productos = await ctx.Productos.ToListAsync();
```

### Error 4: Multiples Enumeraciones

```csharp
// MAL — evalua la consulta dos veces
var query = ctx.Productos.Where(p => p.Activo);
var count = query.Count();      // consulta 1 a SQL
var lista = query.ToList();     // consulta 2 a SQL

// BIEN — materializa una sola vez
var lista  = await ctx.Productos.Where(p => p.Activo).ToListAsync();
var count  = lista.Count;       // O(1) sobre la lista ya cargada
```

### Error 5: Funciones no traducibles en EF Core

```csharp
// MAL — EF Core no puede traducir este metodo a SQL
var resultado = ctx.Productos.Where(p => MiMetodoCustom(p.Precio));

// BIEN — usa expresiones que EF Core pueda traducir
var resultado = ctx.Productos.Where(p => p.Precio > 100 && p.Activo);

// Alternativa: AsEnumerable para evaluacion en cliente (solo si es necesario)
var resultado = ctx.Productos
    .Where(p => p.Activo)
    .AsEnumerable()  // A partir de aqui, se evalua en memoria
    .Where(p => MiMetodoCustom(p.Precio));
```

### Buenas Practicas Resumen

| # | Practica | Ejemplo |
|---|---------|---------|
| 1 | Usa `AsNoTracking()` para solo lectura | `.AsNoTracking().Where(...)` |
| 2 | Proyecta solo campos necesarios | `.Select(p => new { p.Id, p.Nombre })` |
| 3 | Usa `AnyAsync` en vez de `CountAsync` | `.AnyAsync(p => p.Activo)` |
| 4 | Incluye `CancellationToken` | `.ToListAsync(ct)` |
| 5 | Usa `AsSplitQuery` para multiples Includes | `.AsSplitQuery()` |
| 6 | Materializa una sola vez | `.ToListAsync()` una vez, reutiliza la lista |
| 7 | Verifica SQL generado | `.ToQueryString()` |
| 8 | Evita `ToList()` prematuro | Componer primero, materializar al final |

---

## 20. De SQL a LINQ — Guia de Migracion

| SQL | LINQ Method | LINQ Query |
|-----|-------------|------------|
| `SELECT *` | `.Select(p => p)` | `select p` |
| `SELECT col1, col2` | `.Select(p => new { p.Col1, p.Col2 })` | `select new { p.Col1, p.Col2 }` |
| `WHERE cond` | `.Where(p => cond)` | `where cond` |
| `ORDER BY col ASC` | `.OrderBy(p => p.Col)` | `orderby p.Col ascending` |
| `ORDER BY col DESC` | `.OrderByDescending(p => p.Col)` | `orderby p.Col descending` |
| `TOP 5` | `.Take(5)` | No disponible |
| `DISTINCT` | `.Distinct()` | No disponible |
| `COUNT(*)` | `.Count()` | No disponible |
| `SUM(col)` | `.Sum(p => p.Col)` | No disponible |
| `AVG(col)` | `.Average(p => p.Col)` | No disponible |
| `MIN(col)` | `.Min(p => p.Col)` | No disponible |
| `MAX(col)` | `.Max(p => p.Col)` | No disponible |
| `GROUP BY col` | `.GroupBy(p => p.Col)` | `group p by p.Col` |
| `HAVING cond` | `.GroupBy(...).Where(g => cond)` | `group ... into g where cond` |
| `INNER JOIN` | `.Join(...)` | `join ... on ... equals ...` |
| `LEFT JOIN` | `.GroupJoin(...).SelectMany(...DefaultIfEmpty())` | `join ... into ... from ... DefaultIfEmpty()` |
| `OFFSET 10 ROWS FETCH NEXT 5` | `.Skip(10).Take(5)` | No disponible |
| `IN (1,2,3)` | `.Where(p => ids.Contains(p.Id))` | `where ids.Contains(p.Id)` |
| `LIKE '%text%'` | `.Where(p => p.Nombre.Contains("text"))` | `where p.Nombre.Contains("text")` |
| `BETWEEN 10 AND 20` | `.Where(p => p.Precio >= 10 && p.Precio <= 20)` | `where p.Precio >= 10 && p.Precio <= 20` |
| `IS NULL` | `.Where(p => p.Desc == null)` | `where p.Desc == null` |
| `CASE WHEN` | `.Select(p => new { Status = p.Stock > 0 ? "OK" : "No" })` | `select new { Status = p.Stock > 0 ? "OK" : "No" }` |

### Ejemplos de Traduccion Completa

```sql
-- SQL Complejo
SELECT c.Nombre, SUM(v.Monto) AS Total, COUNT(*) AS NumVentas
FROM Ventas v
INNER JOIN Clientes c ON v.ClienteId = c.Id
WHERE v.Fecha >= '2024-01-01' AND v.Monto > 100
GROUP BY c.Nombre
HAVING SUM(v.Monto) > 5000
ORDER BY Total DESC
```

```csharp
// LINQ Equivalente
var resultado = await ctx.Ventas
    .Include(v => v.Cliente)
    .Where(v => v.Fecha >= new DateTime(2024, 1, 1) && v.Monto > 100)
    .GroupBy(v => v.Cliente.Nombre)
    .Select(g => new {
        Nombre = g.Key,
        Total = g.Sum(v => v.Monto),
        NumVentas = g.Count()
    })
    .Where(x => x.Total > 5000)
    .OrderByDescending(x => x.Total)
    .ToListAsync();
```

---

## 21. Ejercicios y Desafios

### Nivel Basico

**Ejercicio 1:** Dada una lista de enteros `{3, 7, 12, 5, 18, 9, 21, 4}`, filtra los numeros mayores a 10 y ordenalos de mayor a menor.

<details>
<summary>Solucion</summary>

```csharp
int[] nums = { 3, 7, 12, 5, 18, 9, 21, 4 };
var resultado = nums.Where(n => n > 10).OrderByDescending(n => n);
// Resultado: 21, 18, 12
```
</details>

**Ejercicio 2:** De una lista de productos, obtiene los nombres de los productos activos con precio menor a 50.

<details>
<summary>Solucion</summary>

```csharp
var nombres = productos
    .Where(p => p.Activo && p.Precio < 50)
    .Select(p => p.Nombre);
```
</details>

### Nivel Intermedio

**Ejercicio 3:** Calcula el valor total del inventario (Precio * Stock) por cada categoria, mostrando solo las categorias con valor mayor a 5000.

<details>
<summary>Solucion</summary>

```csharp
var valorPorCategoria = productos
    .GroupBy(p => p.Categoria)
    .Select(g => new {
        Categoria = g.Key,
        ValorTotal = g.Sum(p => p.Precio * p.Stock)
    })
    .Where(x => x.ValorTotal > 5000)
    .OrderByDescending(x => x.ValorTotal);
```
</details>

**Ejercicio 4:** Usando `Aggregate`, calcula el factorial de 10.

<details>
<summary>Solucion</summary>

```csharp
int factorial = Enumerable.Range(1, 10)
    .Aggregate(1, (acc, n) => acc * n);
// Resultado: 3628800
```
</details>

**Ejercicio 5:** Encuentra los 3 productos mas caros de cada categoria.

<details>
<summary>Solucion</summary>

```csharp
var topPorCategoria = productos
    .GroupBy(p => p.Categoria)
    .Select(g => new {
        Categoria = g.Key,
        Top = g.OrderByDescending(p => p.Precio).Take(3).ToList()
    });
```
</details>

### Nivel Avanzado

**Ejercicio 6:** Implementa un metodo de extension `MinByMaxBy` que encuentre el elemento con el valor minimo y maximo de una propiedad, retornando ambos en una tupla.

<details>
<summary>Solucion</summary>

```csharp
public static (T? Min, T? Max) MinByMaxBy<T, TKey>(
    this IEnumerable<T> source,
    Func<T, TKey> keySelector) where TKey : IComparable<TKey>
{
    var list = source.ToList();
    if (!list.Any()) return (default, default);

    var min = list.Aggregate((a, b) =>
        keySelector(a).CompareTo(keySelector(b)) <= 0 ? a : b);
    var max = list.Aggregate((a, b) =>
        keySelector(a).CompareTo(keySelector(b)) >= 0 ? a : b);

    return (min, max);
}

// Uso
var (masBarato, masCaro) = productos.MinByMaxBy(p => p.Precio);
```
</details>

**Ejercicio 7:** Crea una consulta que calcule la mediana de precios por categoria usando solo LINQ (sin metodos custom).

<details>
<summary>Solucion</summary>

```csharp
var medianaPorCategoria = productos
    .GroupBy(p => p.Categoria)
    .Select(g => new {
        Categoria = g.Key,
        Mediana = g.OrderBy(p => p.Precio)
                   .Skip(g.Count() / 2)
                   .First().Precio
    });
```
</details>

---

## 22. Referencia Rapida

### Operadores de Filtrado
| Metodo | Firma | Descripcion |
|--------|-------|-------------|
| `Where` | `Where(Func<T,bool>)` | Filtra por predicado |
| `OfType<T>` | `OfType<T>()` | Filtra por tipo |

### Operadores de Proyeccion
| Metodo | Firma | Descripcion |
|--------|-------|-------------|
| `Select` | `Select(Func<T,R>)` | Transforma elementos |
| `SelectMany` | `SelectMany(Func<T,IEnum<R>>)` | Aplana colecciones |

### Operadores de Ordenamiento
| Metodo | Firma | Descripcion |
|--------|-------|-------------|
| `OrderBy` | `OrderBy(Func<T,K>)` | Ordena ascendente |
| `OrderByDescending` | `OrderByDescending(Func<T,K>)` | Ordena descendente |
| `ThenBy` | `ThenBy(Func<T,K>)` | Orden secundario asc |
| `ThenByDescending` | `ThenByDescending(Func<T,K>)` | Orden secundario desc |
| `Reverse` | `Reverse()` | Invierte orden |

### Operadores de Agregado
| Metodo | Firma | Descripcion |
|--------|-------|-------------|
| `Count` | `Count()` / `Count(Func<T,bool>)` | Cuenta elementos |
| `LongCount` | `LongCount()` | Cuenta (long) |
| `Sum` | `Sum(Func<T,N>)` | Suma valores |
| `Min` | `Min(Func<T,N>)` | Valor minimo |
| `Max` | `Max(Func<T,N>)` | Valor maximo |
| `Average` | `Average(Func<T,N>)` | Promedio |
| `MinBy` | `MinBy(Func<T,K>)` | Elemento con minimo (.NET 6+) |
| `MaxBy` | `MaxBy(Func<T,K>)` | Elemento con maximo (.NET 6+) |
| `Aggregate` | `Aggregate(seed, func, result)` | Agregado personalizado |

### Operadores de Agrupacion
| Metodo | Firma | Descripcion |
|--------|-------|-------------|
| `GroupBy` | `GroupBy(Func<T,K>)` | Agrupa por clave (diferido) |
| `ToLookup` | `ToLookup(Func<T,K>)` | Agrupa por clave (inmediato) |

### Operadores de Join
| Metodo | Firma | Descripcion |
|--------|-------|-------------|
| `Join` | `Join(inner, outerKey, innerKey, result)` | Inner join |
| `GroupJoin` | `GroupJoin(inner, outerKey, innerKey, result)` | Group join / Left join |
| `Zip` | `Zip(second, result)` | Combina por posicion |

### Operadores de Elemento
| Metodo | Firma | Descripcion |
|--------|-------|-------------|
| `First` | `First()` / `First(Func<T,bool>)` | Primer elemento |
| `FirstOrDefault` | `FirstOrDefault()` | Primero o default |
| `Last` | `Last()` | Ultimo elemento |
| `LastOrDefault` | `LastOrDefault()` | Ultimo o default |
| `Single` | `Single()` | Elemento unico |
| `SingleOrDefault` | `SingleOrDefault()` | Unico o default |
| `ElementAt` | `ElementAt(int)` | Por indice |
| `ElementAtOrDefault` | `ElementAtOrDefault(int)` | Por indice o default |

### Operadores de Cuantificacion
| Metodo | Firma | Descripcion |
|--------|-------|-------------|
| `Any` | `Any()` / `Any(Func<T,bool>)` | Hay alguno? |
| `All` | `All(Func<T,bool>)` | Todos cumplen? |
| `Contains` | `Contains(item)` | Contiene elemento? |

### Operadores de Particion
| Metodo | Firma | Descripcion |
|--------|-------|-------------|
| `Take` | `Take(int)` | Primeros N |
| `Skip` | `Skip(int)` | Saltar N |
| `TakeWhile` | `TakeWhile(Func<T,bool>)` | Tomar mientras |
| `SkipWhile` | `SkipWhile(Func<T,bool>)` | Saltar mientras |

### Operadores de Conjuntos
| Metodo | Firma | Descripcion |
|--------|-------|-------------|
| `Distinct` | `Distinct()` | Sin duplicados |
| `DistinctBy` | `DistinctBy(Func<T,K>)` | Sin duplicados por clave (.NET 6+) |
| `Union` | `Union(second)` | Union sin duplicados |
| `UnionBy` | `UnionBy(second, key)` | Union por clave (.NET 6+) |
| `Intersect` | `Intersect(second)` | Interseccion |
| `IntersectBy` | `IntersectBy(second, key)` | Interseccion por clave (.NET 6+) |
| `Except` | `Except(second)` | Diferencia |
| `ExceptBy` | `ExceptBy(second, key)` | Diferencia por clave (.NET 6+) |
| `Concat` | `Concat(second)` | Concatenar con duplicados |

### Operadores de Conversion
| Metodo | Firma | Descripcion |
|--------|-------|-------------|
| `ToArray` | `ToArray()` | Convertir a array |
| `ToList` | `ToList()` | Convertir a lista |
| `ToDictionary` | `ToDictionary(key, value)` | Convertir a diccionario |
| `ToLookup` | `ToLookup(key, value)` | Convertir a lookup |
| `ToHashSet` | `ToHashSet()` | Convertir a HashSet |
| `AsEnumerable` | `AsEnumerable()` | Bajar a IEnumerable |
| `AsQueryable` | `AsQueryable()` | Subir a IQueryable |
| `Cast<T>` | `Cast<T>()` | Convertir tipos |

### Operadores de Generacion
| Metodo | Firma | Descripcion |
|--------|-------|-------------|
| `Range` | `Enumerable.Range(start, count)` | Rango de enteros |
| `Repeat` | `Enumerable.Repeat(value, count)` | Repetir valor |
| `Empty` | `Enumerable.Empty<T>()` | Secuencia vacia |
| `Chunk` | `Chunk(int size)` | Dividir en lotes (.NET 6+) |
| `DefaultIfEmpty` | `DefaultIfEmpty()` | Secuencia con default si vacia |

### Versiones Async (EF Core)
| Metodo | Descripcion |
|--------|-------------|
| `ToListAsync()` | Materializar async |
| `toArrayAsync()` | Array async |
| `countAsync()` | Contar async |
| `sumAsync(sel)` | Sumar async |
| `averageAsync(sel)` | Promedio async |
| `minAsync(sel)` | Minimo async |
| `maxAsync(sel)` | Maximo async |
| `anyAsync(pred)` | Existe async |
| `firstOrDefaultAsync(pred)` | Primero async |
| `singleOrDefaultAsync(pred)` | Unico async |

---

## Paquetes NuGet Necesarios

```xml
<!-- DAL.csproj -->
<ItemGroup>
  <PackageReference Include="Microsoft.EntityFrameworkCore" Version="8.0.0" />
  <PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="8.0.0" />
  <PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="8.0.0" />
</ItemGroup>

<!-- API / Presentacion.csproj -->
<ItemGroup>
  <PackageReference Include="Microsoft.EntityFrameworkCore.Design" Version="8.0.0" />
  <PackageReference Include="Microsoft.Extensions.DependencyInjection" Version="8.0.0" />
  <PackageReference Include="Microsoft.Extensions.Configuration.Json" Version="8.0.0" />
</ItemGroup>

<!-- Tests.csproj -->
<ItemGroup>
  <PackageReference Include="Microsoft.NET.Test.Sdk" Version="17.8.0" />
  <PackageReference Include="xunit" Version="2.6.2" />
  <PackageReference Include="xunit.runner.visualstudio" Version="2.5.4" />
  <PackageReference Include="FluentAssertions" Version="6.12.0" />
  <PackageReference Include="Microsoft.EntityFrameworkCore.InMemory" Version="8.0.0" />
</ItemGroup>
```

---

> **LINQ en C# — Manual Completo** | .NET 8+ | C# 12 | Visual Studio 2022 | SQL Server | Arquitectura 4 Capas | GitHub Actions
