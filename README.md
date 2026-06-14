# 📘 LINQ en C# — Manual Completo

> **Manual Técnico de Referencia** | Visual Studio Community 2022 | .NET 8+ | C# 12

---

## 📑 Tabla de Contenido

| Cap | Tema | Operadores |
|-----|------|------------|
| 01 | [LINQ Básico — Filtrado y Proyección](#01--linq-básico--filtrado-y-proyección) | `Where`, `Select`, `SelectMany`, `OfType` |
| 02 | [Ordenamiento y Agrupamiento](#02--ordenamiento-y-agrupamiento) | `OrderBy`, `ThenBy`, `OrderByDescending`, `GroupBy`, `ToLookup` |
| 03 | [Operaciones de Join](#03--operaciones-de-join) | `Join`, `GroupJoin`, `Zip` |
| 04 | [Funciones de Agregado](#04--funciones-de-agregado) | `Count`, `LongCount`, `Sum`, `Min`, `Max`, `Average`, `Aggregate` |
| 05 | [Elementos y Cuantificación](#05--elementos-y-cuantificación) | `First`, `FirstOrDefault`, `Last`, `Single`, `Any`, `All`, `Contains` |
| 06 | [Partición y Conjuntos](#06--partición-y-conjuntos) | `Take`, `Skip`, `TakeWhile`, `SkipWhile`, `Distinct`, `Union`, `Intersect`, `Except` |
| 07 | [Conversión y Generación](#07--conversión-y-generación) | `ToArray`, `ToList`, `ToDictionary`, `ToLookup`, `Range`, `Repeat`, `Empty` |
| 08 | [Sintaxis de Consulta (Query Syntax)](#08--sintaxis-de-consulta-query-syntax) | `from`, `where`, `select`, `join`, `group`, `let`, `into` |
| 09 | [LINQ to Objects](#09--linq-to-objects) | `List<T>`, Arrays, Strings, `SelectMany` |
| 10 | [LINQ to XML](#10--linq-to-xml) | `XDocument`, `XElement`, crear/consultar/modificar XML |
| 11 | [Expresiones Lambda y Árboles de Expresión](#11--expresiones-lambda-y-árboles-de-expresión) | `Func<T>`, `Action<T>`, `Expression<T>`, composición |
| 12 | [Ejecución Diferida vs Inmediata](#12--ejecución-diferida-vs-inmediata) | Streaming, Non-streaming, materialización |
| 13 | [PLINQ (Parallel LINQ)](#13--plinq-parallel-linq) | `AsParallel`, `AsOrdered`, `WithCancellation`, `ForAll` |
| 14 | [Patrones Avanzados](#14--patrones-avanzados) | Operadores custom, `PredicateBuilder`, Pivot |
| 15 | [Arquitectura de 4 Capas — Conceptos y Diseño](#15--arquitectura-de-4-capas--conceptos-y-diseño) | Capas, dependencias, diagrama |
| 16 | [Capa de Entidades y Acceso a Datos (DAL)](#16--capa-de-entidades-y-acceso-a-datos-dal) | Entidades, `DbContext`, Repositorio genérico |
| 17 | [Capa de Lógica de Negocio (BLL)](#17--capa-de-lógica-de-negocio-bll) | Servicios, validación, reglas de negocio |
| 18 | [Capa de Presentación (UI)](#18--capa-de-presentación-ui) | Consola, menús, formateo de datos |
| 19 | [Entity Framework Core con SQL Server](#19--entity-framework-core-con-sql-server) | Code-First, Migrations, `Include`, `AsNoTracking` |
| 20 | [Proyecto Integrado — CRUD Completo con 4 Capas](#20--proyecto-integrado--crud-completo-con-4-capas) | CRUD completo, seeding, ejecución |

---

## Estructura de la Solución

```
LINQ_Manual_Completo.sln
├── 01_LINQ_Basico/
│   └── Program.cs
├── 02_Ordenamiento_Agrupamiento/
│   └── Program.cs
├── 03_Joins/
│   └── Program.cs
├── 04_Funciones_Agregado/
│   └── Program.cs
├── 05_Elementos_Cuantificacion/
│   └── Program.cs
├── 06_Particion_Conjuntos/
│   └── Program.cs
├── 07_Conversion_Generacion/
│   └── Program.cs
├── 08_Sintaxis_Consulta/
│   └── Program.cs
├── 09_LINQ_to_Objects/
│   └── Program.cs
├── 10_LINQ_to_XML/
│   └── Program.cs
├── 11_Expresiones_Lambda/
│   └── Program.cs
├── 12_Ejecucion_Diferida_Inmediata/
│   └── Program.cs
├── 13_PLINQ/
│   └── Program.cs
├── 14_Patrones_Avanzados/
│   └── Program.cs
├── 15_Arquitectura_4Capas/
│   ├── Entidades/
│   ├── DAL/
│   ├── BLL/
│   └── Presentacion/
└── 16_EFCore_SQLServer/
    └── Program.cs
```

---

# 01 — LINQ Básico — Filtrado y Proyección

Los operadores de filtrado y proyección constituyen los cimientos fundamentales de LINQ. El filtrado permite seleccionar elementos de una colección que cumplen con una condición específica, mientras que la proyección transforma cada elemento en una nueva forma, ya sea extrayendo propiedades individuales o creando nuevos tipos anónimos con datos calculados. Estos operadores son los más utilizados en cualquier aplicación que manipule datos, y su comprensión profunda es esencial para trabajar eficientemente con LINQ en escenarios reales, desde consultas simples en memoria hasta consultas complejas contra bases de datos SQL Server a través de Entity Framework Core.

## Where — Filtrado de colecciones

El operador `Where` retorna una nueva secuencia que contiene únicamente los elementos que satisfacen la condición especificada en el predicado lambda. Soporta predicados con múltiples condiciones usando operadores lógicos (`&&`, `||`), y existe una sobrecarga que recibe el índice del elemento como segundo parámetro. `Where` es un operador de ejecución diferida (deferred execution), lo que significa que la consulta no se ejecuta hasta que se itera sobre el resultado, ya sea con `foreach`, `ToList()`, o cualquier otra operación de materialización.

```csharp
// Where: Productos con precio mayor a $50
var caros = productos.Where(p => p.Precio > 50);

// Where con condiciones múltiples
var accesorios = productos.Where(p =>
    p.Categoria == "Accesorio" && p.Stock < 50);

// Where con índice (segundo parámetro = índice del elemento)
var primeros5Caros = productos
    .Where((p, index) => p.Precio > 100 && index < 10);

// Where con operador OR
var busqueda = productos.Where(p =>
    p.Nombre.Contains("Laptop") || p.Nombre.Contains("PC"));
```

**Salida de consola:**
```
► Where: Productos con precio mayor a $50
  Laptop HP             | Computo        | $1200.00 | Stock: 15
  Teclado Mecanico      | Accesorio      | $80.00   | Stock: 45
  Monitor Samsung       | Computo        | $450.00  | Stock: 8
  RAM DDR4 16GB         | Almacenamiento | $75.00   | Stock: 50
  Impresora Epson       | Impresion      | $280.00  | Stock: 12
  Auriculares Sony      | Accesorio      | $120.00  | Stock: 25

► Where: Accesorios con stock bajo
  Teclado Mecanico      | Accesorio      | $80.00   | Stock: 45
```

> 💡 **Tip:** `Where` no modifica la colección original. Siempre retorna una nueva secuencia `IEnumerable<T>`. Para evaluar la condición, usa siempre una expresión lambda que retorne `bool`.

> ⚠️ **Precaución:** Si aplicas múltiples `Where` consecutivos, LINQ los optimiza internamente en un solo filtro cuando es posible, pero para claridad del código, es preferible combinar las condiciones con `&&` en un solo `Where`.

## Select — Proyección

El operador `Select` transforma cada elemento de la secuencia de entrada en una nueva forma. Puede proyectar a un tipo anónimo (usando `new { }`), seleccionar propiedades individuales, aplicar cálculos sobre los datos, o incluso transformar a un tipo concreto diferente. Al igual que `Where`, `Select` es de ejecución diferida. También dispone de una sobrecarga con índice que permite acceder a la posición del elemento durante la proyección, lo cual es útil para numerar resultados o aplicar lógica basada en la posición.

```csharp
// Select: Proyección simple — solo nombres
var nombres = productos.Select(p => p.Nombre);

// Select: Proyección a tipo anónimo (con IVA calculado)
var conIva = productos.Select(p => new {
    p.Nombre,
    PrecioBase = p.Precio,
    PrecioIVA = p.Precio * 1.16m,
    Descuento = p.Precio > 500 ? p.Precio * 0.10m : 0
});

// Select con índice
var indexados = productos.Select((p, i) =>
    $"#{i + 1} {p.Nombre} - ${p.Precio:F2}");

// Select: Proyección a tipo concreto
class ProductoDTO {
    public string Nombre { get; set; }
    public decimal PrecioConIVA { get; set; }
}

var dtos = productos.Select(p => new ProductoDTO {
    Nombre = p.Nombre,
    PrecioConIVA = p.Precio * 1.16m
}).ToList();
```

**Salida de consola:**
```
► Select: Solo nombres
  Laptop HP, Mouse Logitech, Teclado Mecanico, Monitor Samsung...

► Select: Precio con IVA
  Laptop HP             | Base: $1200.00 | IVA: $1392.00 | Desc: $120.00
  Mouse Logitech        | Base: $35.00   | IVA: $40.60   | Desc: $0.00
  Monitor Samsung       | Base: $450.00  | IVA: $522.00  | Desc: $45.00

► Select con índice
  #1 Laptop HP - $1200.00
  #2 Mouse Logitech - $35.00
  #3 Teclado Mecanico - $80.00
```

## SelectMany — Aplanar colecciones

`SelectMany` es uno de los operadores más poderosos y subestimados de LINQ. Mientras que `Select` produce una secuencia de secuencias (uno-a-uno), `SelectMany` aplana colecciones anidadas en una secuencia única (uno-a-muchos). Es esencial cuando se trabaja con listas dentro de objetos, como las categorías de un producto, los pedidos de un cliente, o los tags de un artículo. En sintaxis de consulta, `SelectMany` se expresa con múltiples cláusulas `from`, lo que lo hace más legible en consultas complejas.

```csharp
// SelectMany: Aplanar lista de listas
var categorias = new List<List<string>> {
    new() { "Laptop", "PC", "Tablet" },
    new() { "Mouse", "Teclado" },
    new() { "SSD", "RAM", "USB" }
};
var todos = categorias.SelectMany(cat => cat);
// Resultado: Laptop, PC, Tablet, Mouse, Teclado, SSD, RAM, USB

// SelectMany con resultado en tipo anónimo
var productosConCategorias = productos
    .SelectMany(p => p.Tags, (producto, tag) => new {
        Producto = producto.Nombre,
        Tag = tag
    });

// SelectMany: Obtener todos los pedidos de todos los clientes
var todosLosPedidos = clientes.SelectMany(c => c.Pedidos);

// SelectMany con filtro combinado
var pedidosGrandes = clientes
    .SelectMany(c => c.Pedidos)
    .Where(p => p.Total > 1000)
    .OrderByDescending(p => p.Total);
```

**Salida de consola:**
```
► SelectMany: Todas las categorías aplanadas
  Laptop, PC, Tablet, Mouse, Teclado, SSD, RAM, USB

► SelectMany: Productos con sus tags
  Laptop HP       | Tag: computo
  Laptop HP       | Tag: premium
  Mouse Logitech  | Tag: accesorio
  Mouse Logitech  | Tag: economico
```

> 💡 **Tip:** `SelectMany` es equivalente a `flatMap` en otros lenguajes como JavaScript o Java. En Query Syntax, se escribe como múltiples cláusulas `from`.

## OfType — Filtrar por tipo

El operador `OfType<T>` filtra los elementos de una colección heterogénea (tipicamente `IEnumerable<object>` o `ArrayList`) y retorna solo los que son del tipo especificado. A diferencia de `Cast<T>`, que lanza una excepción si un elemento no se puede convertir, `OfType<T>` simplemente omite los elementos que no coinciden con el tipo. Esto lo hace ideal para trabajar con colecciones legacy o APIs que retornan `object`, como ciertas colecciones de Windows Forms o datos deserializados dinámicamente.

```csharp
// OfType: Filtrar enteros de una colección mixta
object[] mixta = { 1, "hola", 3.14, 42, true, "mundo", 7 };
var enteros = mixta.OfType<int>();      // Resultado: 1, 42, 7
var cadenas = mixta.OfType<string>();    // Resultado: "hola", "mundo"

// OfType con controles Windows Forms
var botones = this.Controls.OfType<Button>();
var textBoxes = this.Controls.OfType<TextBox>();

// OfType + Where combinado
var enterosGrandes = mixta.OfType<int>().Where(x => x > 10);
// Resultado: 42
```

> ⚠️ **Nota:** `OfType` solo filtra tipos compatibles en tiempo de ejecución. No realiza conversiones explícitas. Para conversiones, usa `Cast<T>` seguido de `Select`.

### Tabla resumen — Operadores básicos

| Operador | Tipo | Ejecución | Descripción |
|----------|------|-----------|-------------|
| `Where` | Filtrado | Diferida | Filtra elementos basándose en un predicado |
| `Select` | Proyección | Diferida | Transforma cada elemento en una nueva forma |
| `SelectMany` | Proyección | Diferida | Aplana colecciones anidadas en una secuencia |
| `OfType<T>` | Filtrado | Diferida | Filtra elementos por tipo en colecciones heterogéneas |
| `Cast<T>` | Conversión | Diferida | Convierte todos los elementos al tipo especificado |

---

# 02 — Ordenamiento y Agrupamiento

Los operadores de ordenamiento permiten clasificar los datos de forma ascendente o descendente, con soporte para múltiples criterios mediante `ThenBy` y `ThenByDescending`. Los operadores de agrupamiento organizan los elementos en grupos basados en una clave común, lo que es fundamental para generar reportes, estadísticas y resúmenes de datos. La combinación de ordenamiento y agrupamiento con funciones de agregado permite crear consultas analíticas poderosas similares a las que se realizan con `GROUP BY` en SQL Server.

## OrderBy / OrderByDescending / ThenBy / ThenByDescending

`OrderBy` ordena los elementos de la secuencia en orden ascendente según la clave especificada, mientras que `OrderByDescending` lo hace en orden descendente. Para ordenamientos múltiples (equivalente a `ORDER BY Col1, Col2` en SQL), se usa `ThenBy` y `ThenByDescending` después del `OrderBy` inicial. Es importante notar que si se encadenan múltiples `OrderBy`, cada uno reemplaza al anterior; para ordenamientos múltiples siempre debe usarse `ThenBy` después del primer `OrderBy`.

```csharp
// OrderBy: Orden ascendente por salario
var porSalario = empleados.OrderBy(e => e.Salario);

// OrderByDescending: Orden descendente por antigüedad
var porAntiguedad = empleados.OrderByDescending(e => e.Antiguedad);

// ThenBy: Ordenamiento múltiple (equivalente a ORDER BY Dept, Salario)
var multiOrden = empleados
    .OrderBy(e => e.Departamento)
    .ThenBy(e => e.Salario);

// Ordenamiento con condición personalizada
var porPrioridad = tareas
    .OrderBy(t => t.Prioridad == "Alta" ? 0 :
                  t.Prioridad == "Media" ? 1 : 2)
    .ThenBy(t => t.FechaLimite);

// Ordenamiento por múltiples campos
var completo = empleados
    .OrderBy(e => e.Departamento)
    .ThenByDescending(e => e.Salario)
    .ThenBy(e => e.Nombre);
```

**Salida de consola:**
```
► OrderBy: Empleados ordenados por salario (ascendente)
  Laura Sanchez     | RRHH         | $3200.00
  Diego Ramirez     | RRHH         | $3400.00
  Ana Garcia        | Ventas       | $3500.00
  Pedro Martinez    | Ventas       | $3800.00
  Carlos Lopez      | Finanzas     | $4200.00
  Maria Rodriguez   | TI           | $4500.00
  Jorge Fernandez   | TI           | $5500.00

► ThenBy: Por departamento, luego por salario
  Finanzas | Carlos Lopez     | $4200.00
  RRHH     | Laura Sanchez    | $3200.00
  RRHH     | Diego Ramirez    | $3400.00
  TI       | Maria Rodriguez  | $4500.00
  TI       | Jorge Fernandez  | $5500.00
  Ventas   | Ana Garcia       | $3500.00
  Ventas   | Pedro Martinez   | $3800.00
```

> ⚠️ **Error común:** No uses `OrderBy` encadenados para ordenar por múltiples campos. Cada `OrderBy` nuevo resetea el ordenamiento. Usa `ThenBy` después del primer `OrderBy`.

## GroupBy — Agrupamiento

`GroupBy` organiza los elementos en grupos basados en una clave. Cada grupo es un objeto `IGrouping<TKey, TElement>` que contiene la propiedad `Key` (la clave del grupo) y es iterable sobre los elementos del grupo. Se puede combinar con proyecciones para obtener estadísticas por grupo (promedio, suma, máximo, mínimo), lo cual es equivalente a las consultas `GROUP BY` con funciones de agregado en SQL Server. También soporta claves compuestas usando tipos anónimos.

```csharp
// GroupBy: Agrupar por departamento
var grupos = empleados.GroupBy(e => e.Departamento);

foreach (var grupo in grupos) {
    Console.WriteLine($"Departamento: {grupo.Key}");
    foreach (var emp in grupo)
        Console.WriteLine($"  {emp.Nombre} - ${emp.Salario}");
}

// GroupBy + Select: Estadísticas por grupo
var salariosPromedio = empleados
    .GroupBy(e => e.Departamento)
    .Select(g => new {
        Departamento = g.Key,
        Cantidad = g.Count(),
        Promedio = g.Average(e => e.Salario),
        Maximo = g.Max(e => e.Salario),
        Minimo = g.Min(e => e.Salario),
        Total = g.Sum(e => e.Salario)
    });

// GroupBy con clave compuesta
var porDeptoRango = empleados.GroupBy(e => new {
    e.Departamento,
    Rango = e.Salario >= 4500 ? "Alto" : "Medio"
});

// GroupBy con proyección de elementos
var nombresPorDepto = empleados
    .GroupBy(e => e.Departamento, e => e.Nombre);
```

**Salida de consola:**
```
► GroupBy: Salario promedio por departamento
  Finanzas    | 2 emp | Prom: $4350.00 | Max: $4500.00 | Min: $4200.00 | Total: $8700.00
  RRHH        | 2 emp | Prom: $3300.00 | Max: $3400.00 | Min: $3200.00 | Total: $6600.00
  TI          | 3 emp | Prom: $5166.67 | Max: $5500.00 | Min: $4500.00 | Total: $15500.00
  Ventas      | 3 emp | Prom: $3633.33 | Max: $3800.00 | Min: $3500.00 | Total: $10900.00

► GroupBy: Clave compuesta (Depto + Rango)
  Finanzas | Alto  | Carlos Lopez  | $4500.00
  Finanzas | Medio | Ana Torres    | $4200.00
  TI       | Alto  | Jorge Fernandez | $5500.00
```

## ToLookup — Agrupación inmediata

`ToLookup` es similar a `GroupBy` pero ejecuta la consulta inmediatamente y retorna un `ILookup<TKey, TElement>`, que es un diccionario inmutable de solo lectura donde cada clave puede tener múltiples valores asociados. La diferencia clave con `GroupBy` es que `GroupBy` usa ejecución diferida mientras que `ToLookup` materializa los resultados en el momento. `ILookup` es ideal cuando necesitas acceder repetidamente a los grupos por clave, ya que la búsqueda es O(1) a diferencia de iterar sobre `IGrouping`.

```csharp
// ToLookup: Agrupación inmediata
var lookup = empleados.ToLookup(e => e.Departamento);

// Acceso directo por clave O(1)
if (lookup.Contains("TI")) {
    var empTI = lookup["TI"];
    foreach (var emp in empTI)
        Console.WriteLine($"  {emp.Nombre} - ${emp.Salario}");
}

// ToLookup con proyección
var nombresLookup = empleados
    .ToLookup(e => e.Departamento, e => e.Nombre);

// ToLookup con clave compuesta
var compuesto = empleados.ToLookup(e =>
    e.Departamento + " - " + (e.Salario >= 4000 ? "Senior" : "Junior"));
```

> 💡 **Tip:** Usa `ToLookup` cuando necesites acceder a los grupos por clave múltiples veces. Usa `GroupBy` cuando solo necesites iterar sobre todos los grupos una vez.

### Tabla resumen — Ordenamiento y Agrupamiento

| Operador | Tipo | Ejecución | Descripción |
|----------|------|-----------|-------------|
| `OrderBy` | Ordenamiento | Diferida | Ordena ascendente por clave |
| `OrderByDescending` | Ordenamiento | Diferida | Ordena descendente por clave |
| `ThenBy` | Ordenamiento | Diferida | Orden secundario ascendente |
| `ThenByDescending` | Ordenamiento | Diferida | Orden secundario descendente |
| `Reverse` | Ordenamiento | Diferida | Invierte el orden de la secuencia |
| `GroupBy` | Agrupamiento | Diferida | Agrupa por clave (deferred) |
| `ToLookup` | Agrupamiento | Inmediata | Agrupa por clave (materializado) |

---

# 03 — Operaciones de Join

Las operaciones de join en LINQ permiten combinar datos de dos secuencias diferentes basándose en una clave común, de manera similar a los `JOIN` en SQL Server. LINQ ofrece tres operadores principales de join: `Join` para inner joins, `GroupJoin` para group joins (equivalente a LEFT JOIN con agrupamiento), y `Zip` para combinar secuencias por posición. Comprender estos operadores es esencial para trabajar con datos relacionales en memoria y para optimizar consultas contra bases de datos donde las relaciones entre tablas son fundamentales.

## Join — Inner Join

El operador `Join` realiza un inner join entre dos secuencias basándose en claves iguales. Solo incluye los elementos que tienen coincidencia en ambas secuencias, descartando los que no tienen par. Es equivalente a `INNER JOIN` en SQL Server. La sintaxis de método recibe cuatro parámetros: la secuencia externa, la función de clave externa, la función de clave interna, y la función de proyección del resultado.

```csharp
// Join: Productos con sus categorías
var productosConCategoria = productos
    .Join(categorias,
        p => p.CategoriaId,        // Clave externa
        c => c.Id,                  // Clave interna
        (p, c) => new {            // Proyección del resultado
            p.Nombre,
            p.Precio,
            Categoria = c.Nombre
        });

// Join: Empleados con sus departamentos
var empDepto = empleados
    .Join(departamentos,
        e => e.DepartamentoId,
        d => d.Id,
        (e, d) => new {
            e.Nombre,
            e.Salario,
            Departamento = d.Nombre,
            d.Ubicacion
        });

// Join con clave compuesta
var resultado = ordenes
    .Join(clientes,
        o => new { o.ClienteId, o.Sucursal },
        c => new { c.Id, c.Sucursal },
        (o, c) => new { o.Numero, c.Nombre, o.Total });
```

**Salida de consola:**
```
► Join: Productos con categoría
  Laptop HP       | $1200.00 | Categoría: Computo
  Mouse Logitech  | $35.00   | Categoría: Accesorio
  Monitor Samsung | $450.00  | Categoría: Computo

► Join: Empleados con departamento
  Jorge Fernandez | $5500.00 | TI - Piso 3
  Maria Rodriguez | $4500.00 | TI - Piso 3
```

## GroupJoin — Left Join con agrupamiento

`GroupJoin` realiza un group join que es equivalente a un LEFT OUTER JOIN en SQL, pero en lugar de producir filas duplicadas para la tabla externa, agrupa los resultados internos en una colección por cada elemento externo. Si un elemento externo no tiene coincidencias internas, su colección estará vacía. Para convertir un `GroupJoin` en un verdadero LEFT JOIN plano, se usa `SelectMany` con `DefaultIfEmpty()`.

```csharp
// GroupJoin: Categorías con sus productos
var categoriasConProductos = categorias
    .GroupJoin(productos,
        c => c.Id,
        p => p.CategoriaId,
        (c, prods) => new {
            Categoria = c.Nombre,
            Productos = prods,
            Cantidad = prods.Count()
        });

// GroupJoin + SelectMany + DefaultIfEmpty = LEFT JOIN
var leftJoin = categorias
    .GroupJoin(productos,
        c => c.Id,
        p => p.CategoriaId,
        (c, prods) => new { c, prods })
    .SelectMany(x => x.prods.DefaultIfEmpty(),
        (x, p) => new {
            Categoria = x.c.Nombre,
            Producto = p?.Nombre ?? "SIN PRODUCTO",
            Precio = p?.Precio ?? 0
        });
```

**Salida de consola:**
```
► GroupJoin: Categorías con productos
  Computo    | 3 productos
  Accesorio  | 2 productos
  Impresion  | 1 producto
  Redes      | 0 productos

► LEFT JOIN: Todas las categorías (con o sin productos)
  Computo    | Laptop HP        | $1200.00
  Computo    | Monitor Samsung  | $450.00
  Redes      | SIN PRODUCTO     | $0.00
```

## Zip — Combinación por posición

El operador `Zip` combina dos secuencias elemento por elemento basándose en su posición (índice), produciendo una nueva secuencia donde cada elemento es el resultado de aplicar una función a los elementos correspondientes de ambas secuencias. Si las secuencias tienen diferente longitud, `Zip` se detiene cuando la secuencia más corta se agota.

```csharp
// Zip: Combinar nombres con salarios
var nombres = new[] { "Ana", "Carlos", "Maria" };
var salarios = new[] { 3500m, 4200m, 5500m };

var combinados = nombres.Zip(salarios, (nombre, salario) =>
    $"{nombre} gana ${salario:F2}");

// Zip con tres secuencias (C# 12+)
var ids = new[] { 1, 2, 3 };
var resultado = ids.Zip(nombres, salarios)
    .Select(x => $"ID:{x.First} | {x.Second} | ${x.Third:F2}");
```

---

# 04 — Funciones de Agregado

Las funciones de agregado calculan un valor único a partir de una colección de valores. Son esenciales para generar reportes, estadísticas y resúmenes de datos en cualquier aplicación empresarial. En el contexto de una arquitectura de 4 capas con SQL Server, las funciones de agregado se utilizan principalmente en la Capa de Lógica de Negocio (BLL) para calcular totales, promedios, y métricas que luego se presentan al usuario. LINQ proporciona funciones de agregado estándar (`Count`, `Sum`, `Min`, `Max`, `Average`) y el operador genérico `Aggregate` para cálculos personalizados.

## Count y LongCount

`Count` retorna el número de elementos en una secuencia, con o sin condición. `LongCount` hace lo mismo pero retorna `long` en lugar de `int`, lo cual es necesario para colecciones con más de 2,147,483,647 elementos. Ambos tienen una sobrecarga sin parámetro que simplemente cuenta todos los elementos, y una sobrecarga con predicado que cuenta solo los que cumplen la condición.

```csharp
// Count: Total de productos
int total = productos.Count();

// Count con condición
int caros = productos.Count(p => p.Precio > 100);

// LongCount para colecciones grandes
long totalGrande = datosGrandes.LongCount();

// Count en GroupBy
var porCategoria = productos
    .GroupBy(p => p.Categoria)
    .Select(g => new {
        Categoria = g.Key,
        Cantidad = g.Count()
    });
```

## Sum, Min, Max, Average

Estas funciones calculan la suma, mínimo, máximo y promedio de valores numéricos en una secuencia. Cada una acepta un selector que indica qué propiedad numérica se debe agregar. Sin el selector, operan directamente sobre secuencias de tipos numéricos (`int`, `decimal`, `double`, etc.).

```csharp
// Sum: Total del inventario
decimal totalInventario = productos.Sum(p => p.Precio * p.Stock);

// Min / Max: Rango de precios
decimal precioMin = productos.Min(p => p.Precio);
decimal precioMax = productos.Max(p => p.Precio);

// Average: Salario promedio por departamento
var promedios = empleados
    .GroupBy(e => e.Departamento)
    .Select(g => new {
        Departamento = g.Key,
        Promedio = g.Average(e => e.Salario)
    });

// Sum con condición previa
decimal totalCaros = productos
    .Where(p => p.Precio > 100)
    .Sum(p => p.Precio);

// Min/Max con tipo anónimo
var estadisticas = productos
    .GroupBy(p => p.Categoria)
    .Select(g => new {
        Categoria = g.Key,
        PrecioMin = g.Min(p => p.Precio),
        PrecioMax = g.Max(p => p.Precio),
        StockTotal = g.Sum(p => p.Stock),
        Promedio = g.Average(p => p.Precio)
    });
```

**Salida de consola:**
```
► Estadísticas por categoría
  Computo        | Min: $35.00  | Max: $1200.00 | Stock: 68  | Prom: $561.67
  Accesorio      | Min: $25.00  | Max: $120.00  | Stock: 120 | Prom: $75.00
  Almacenamiento | Min: $45.00  | Max: $75.00   | Stock: 150 | Prom: $60.00
  Impresion      | Min: $280.00 | Max: $280.00  | Stock: 12  | Prom: $280.00
```

## Aggregate — Agregado personalizado

`Aggregate` es el operador de agregado más flexible. Aplica una función acumuladora sobre la secuencia, comenzando con un valor semilla opcional, y produce un resultado final. Es útil para cálculos personalizados que no se pueden expresar con `Sum`, `Min`, `Max` o `Average`, como concatenaciones con formato, cálculos de desviación estándar, o acumulaciones complejas con estado.

```csharp
// Aggregate: Concatenar nombres con coma
var nombresConcat = productos
    .Select(p => p.Nombre)
    .Aggregate((acc, nombre) => acc + ", " + nombre);

// Aggregate con valor semilla
var totalConIVA = productos
    .Aggregate(0m, (total, p) => total + p.Precio * 1.16m);

// Aggregate con semilla y selector de resultado
var stats = productos.Aggregate(
    new { Min = decimal.MaxValue, Max = 0m, Total = 0m, Count = 0 },
    (acc, p) => new {
        Min = Math.Min(acc.Min, p.Precio),
        Max = Math.Max(acc.Max, p.Precio),
        Total = acc.Total + p.Precio,
        Count = acc.Count + 1
    },
    result => new {
        result.Min,
        result.Max,
        Promedio = result.Total / result.Count
    });

// Aggregate: Cálculo de desviación estándar
var desvStd = ventas.Select(v => (double)v.Monto)
    .Aggregate(
        new { Sum = 0.0, SumSq = 0.0, Count = 0 },
        (acc, val) => new {
            Sum = acc.Sum + val,
            SumSq = acc.SumSq + val * val,
            Count = acc.Count + 1
        },
        r => Math.Sqrt(r.SumSq / r.Count - Math.Pow(r.Sum / r.Count, 2)));
```

> 💡 **Tip:** `Aggregate` puede reemplazar bucles `foreach` con acumuladores. Sin embargo, para operaciones simples como suma o concatenación, prefiere `Sum` o `string.Join()` que son más legibles.

---

# 05 — Elementos y Cuantificación

Los operadores de elementos retornan un elemento específico de la secuencia, mientras que los operadores de cuantificación evalúan condiciones sobre la colección y retornan un valor booleano. Los operadores de elementos son de ejecución inmediata y pueden lanzar excepciones si no encuentran el elemento buscado (versión sin `OrDefault`), o retornar el valor por defecto del tipo (versión con `OrDefault`). Los operadores de cuantificación son extremadamente útiles para validaciones en la capa de negocio.

## First / FirstOrDefault / Last / LastOrDefault

```csharp
// First: Primer elemento (lanza excepción si está vacía)
var primerProducto = productos.First();

// First con condición
var primerCaro = productos.First(p => p.Precio > 500);

// FirstOrDefault: Retorna null si no encuentra
var primerElectronica = productos.FirstOrDefault(p =>
    p.Categoria == "Electronica");  // null si no existe

// Last / LastOrDefault
var ultimoEmpleado = empleados.Last();
var ultimoConAltoSalario = empleados
    .LastOrDefault(e => e.Salario > 5000);
```

## Single / SingleOrDefault

`Single` retorna el único elemento de la secuencia y lanza excepción si hay más de uno o ninguno. `SingleOrDefault` retorna `default` si no hay elementos. Son ideales para validaciones donde esperas exactamente un resultado.

```csharp
// Single: Exactamente un elemento
var producto = productos.Single(p => p.Id == 5);

// SingleOrDefault: Uno o ninguno
var admin = usuarios.SingleOrDefault(u => u.Rol == "Admin");
```

> ⚠️ **Precaución:** `Single` lanza `InvalidOperationException` si la secuencia tiene más de un elemento. Úsalo solo cuando estés seguro de que debe haber exactamente uno.

## Any / All / Contains

```csharp
// Any: ¿Hay algún elemento?
bool hayProductos = productos.Any();

// Any con condición: ¿Hay productos caros?
bool hayCaros = productos.Any(p => p.Precio > 1000);

// All: ¿Todos cumplen la condición?
bool todosActivos = productos.All(p => p.Activo);

// Contains: ¿Contiene el elemento?
bool existe = productos.Contains(productoEspecifico);

// Contains con comparer personalizado
bool existeNombre = nombres.Select(n => n.ToUpper())
    .Contains("LAPTOP HP");
```

### Tabla resumen — Elementos y Cuantificación

| Operador | Retorna | Si no existe | Ejecución |
|----------|---------|-------------|-----------|
| `First` | `T` | Excepción | Inmediata |
| `FirstOrDefault` | `T?` | `default/null` | Inmediata |
| `Last` | `T` | Excepción | Inmediata |
| `LastOrDefault` | `T?` | `default/null` | Inmediata |
| `Single` | `T` | Excepción | Inmediata |
| `SingleOrDefault` | `T?` | `default/null` | Inmediata |
| `Any` | `bool` | `false` | Inmediata |
| `All` | `bool` | `true` (vacía) | Inmediata |
| `Contains` | `bool` | `false` | Inmediata |

---

# 06 — Partición y Conjuntos

Los operadores de partición dividen una secuencia en partes sin reordenar los elementos, mientras que los operadores de conjuntos realizan operaciones de teoría de conjuntos sobre dos secuencias. Los operadores de partición son fundamentales para implementar paginación en aplicaciones con grandes volúmenes de datos, especialmente en la capa de presentación donde se muestran resultados página por página.

## Take / Skip / TakeWhile / SkipWhile

```csharp
// Take: Primeros N elementos
var top5 = productos.OrderByDescending(p => p.Precio).Take(5);

// Skip: Saltar N elementos
var sinPrimeros3 = productos.Skip(3);

// Paginación: Take + Skip
int pagina = 2, tamaño = 10;
var pagina2 = productos
    .Skip((pagina - 1) * tamaño)
    .Take(tamaño);

// TakeWhile: Tomar mientras se cumpla la condición
var hastaCien = productos.OrderBy(p => p.Precio)
    .TakeWhile(p => p.Precio < 100);

// SkipWhile: Saltar mientras se cumpla
var despuesDeBaratos = productos.OrderBy(p => p.Precio)
    .SkipWhile(p => p.Precio < 50);
```

## Distinct / Union / Intersect / Except

```csharp
// Distinct: Eliminar duplicados
var categorias = productos.Select(p => p.Categoria).Distinct();

// Distinct con comparador personalizado
var unicos = productos.DistinctBy(p => p.Categoria);

// Union: Combinar sin duplicados
var todos = listaA.Union(listaB);

// Intersect: Elementos en común
var comunes = listaA.Intersect(listaB);

// Except: Elementos en A pero no en B
var soloEnA = listaA.Except(listaB);
```

---

# 07 — Conversión y Generación

Los operadores de conversión transforman una secuencia en un tipo de colección específico, forzando la ejecución inmediata de la consulta. Los operadores de generación crean nuevas secuencias a partir de parámetros, sin necesidad de una colección fuente. En una arquitectura de 4 capas, los operadores de conversión se usan frecuentemente para materializar resultados antes de transferirlos entre capas, mientras que los de generación son útiles para crear datos de prueba y testing.

## ToArray / ToList / ToDictionary / ToLookup

```csharp
// ToList: Materializar en List<T>
List<Producto> lista = productos.Where(p => p.Activo).ToList();

// ToArray: Materializar en array
Producto[] array = productos.OrderBy(p => p.Precio).ToArray();

// ToDictionary: Clave-valor (clave única)
var dict = productos.ToDictionary(p => p.Id, p => p.Nombre);

// ToDictionary con selector de elemento completo
var dictCompleto = productos.ToDictionary(p => p.Id);

// ToHashSet: Conjunto sin duplicados
var hashSet = productos.Select(p => p.Categoria).ToHashSet();
```

## Range / Repeat / Empty

```csharp
// Range: Generar secuencia de números
var numeros = Enumerable.Range(1, 100);  // 1, 2, 3, ..., 100

// Range para generar años
var años = Enumerable.Range(2020, 7)  // 2020-2026
    .Select(a => new { Año = a, EsBisiesto = DateTime.IsLeapYear(a) });

// Repeat: Repetir un valor
var ceros = Enumerable.Repeat(0, 10);  // [0,0,0,0,0,0,0,0,0,0]

// Empty: Secuencia vacía tipada
var vacio = Enumerable.Empty<Producto>();
```

> 💡 **Tip:** Usa `ToList()` o `ToArray()` para materializar consultas antes de iterar múltiples veces o antes de transferir datos entre capas. Una consulta diferida se ejecuta cada vez que se itera.

---

# 08 — Sintaxis de Consulta (Query Syntax)

La sintaxis de consulta (Query Syntax) es una alternativa declarativa a la sintaxis de métodos (Method Syntax) que se asemeja más a SQL. Ambas sintaxis son equivalentes y se compilan al mismo IL, pero la Query Syntax es más legible para consultas complejas con joins, agrupamientos y proyecciones. En una arquitectura de 4 capas, la Query Syntax puede hacer las consultas más fáciles de revisar y mantener en la Capa de Acceso a Datos.

## from / where / select / orderby

```csharp
// Query Syntax básica
var caros = from p in productos
            where p.Precio > 100
            orderby p.Precio descending
            select p;

// Proyección con tipo anónimo
var resumen = from p in productos
              where p.Stock > 0
              orderby p.Categoria, p.Precio
              select new {
                  p.Nombre,
                  p.Categoria,
                  Total = p.Precio * p.Stock
              };
```

## join / group / let / into

```csharp
// join en Query Syntax
var conCategoria = from p in productos
                   join c in categorias on p.CategoriaId equals c.Id
                   select new { p.Nombre, Categoria = c.Nombre };

// group en Query Syntax
var porCategoria = from p in productos
                   group p by p.Categoria into grupo
                   select new {
                       Categoria = grupo.Key,
                       Cantidad = grupo.Count(),
                       Total = grupo.Sum(p => p.Precio)
                   };

// let para variables intermedias
var conDescuento = from p in productos
                   let precioConIVA = p.Precio * 1.16m
                   let descuento = precioConIVA > 500 ? 0.10m : 0.05m
                   select new {
                       p.Nombre,
                       PrecioFinal = precioConIVA * (1 - descuento)
                   };

// into para continuaciones
var grupos = from p in productos
             group p by p.Categoria into g
             where g.Count() > 2
             select new { Categoria = g.Key, Total = g.Count() };
```

---

# 09 — LINQ to Objects

LINQ to Objects es el proveedor más fundamental de LINQ, que permite consultar cualquier colección en memoria que implemente `IEnumerable<T>`. Esto incluye `List<T>`, arrays, `Dictionary<TKey,TValue>`, strings, y cualquier colección personalizada. A diferencia de LINQ to SQL o EF Core, LINQ to Objects ejecuta todas las operaciones en memoria sin traducir a SQL, lo que lo hace ideal para datos ya cargados o para lógica de negocio pura.

## Consultas sobre List<T> y Arrays

```csharp
// Filtrar lista de productos
var activos = productos.Where(p => p.Activo).ToList();

// Buscar en array
int[] numeros = { 5, 12, 3, 28, 7, 15, 42, 9 };
var mayores10 = numeros.Where(n => n > 10).OrderBy(n => n);
// Resultado: 12, 15, 28, 42

// Operaciones sobre strings
string texto = "LINQ es poderoso y flexible";
var palabras = texto.Split(' ')
    .Where(p => p.Length > 3)
    .OrderBy(p => p.Length);
// Resultado: "LINQ", "poderoso", "flexible"

// LINQ con Dictionary
var dict = new Dictionary<string, decimal> {
    ["Laptop"] = 1200m, ["Mouse"] = 35m, ["Monitor"] = 450m
};
var caros = dict.Where(kvp => kvp.Value > 100)
    .Select(kvp => $"{kvp.Key}: ${kvp.Value}");
```

## SelectMany con colecciones anidadas

```csharp
// Obtener todos los pedidos de todos los clientes
var todosPedidos = clientes.SelectMany(c => c.Pedidos);

// Con proyección
var detalle = clientes.SelectMany(
    c => c.Pedidos,
    (cliente, pedido) => new {
        Cliente = cliente.Nombre,
        pedido.Numero,
        pedido.Total
    });
```

---

# 10 — LINQ to XML

LINQ to XML es un proveedor que permite crear, consultar y modificar documentos XML usando la misma sintaxis LINQ que se usa para colecciones en memoria. Utiliza las clases `XDocument`, `XElement`, `XAttribute` del namespace `System.Xml.Linq`, que proporcionan un modelo de programación más simple y funcional que el DOM tradicional. En una arquitectura de 4 capas, LINQ to XML se usa frecuentemente en la capa de integración para consumir o generar servicios XML.

## Crear y consultar XML

```csharp
// Crear documento XML con LINQ
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
```

---

# 11 — Expresiones Lambda y Árboles de Expresión

Las expresiones lambda son funciones anónimas que se usan extensamente en LINQ como argumentos de operadores de consulta. Los árboles de expresión (`Expression<T>`) representan código como una estructura de datos que puede ser analizada, modificada y traducida en tiempo de ejecución, lo cual es la base sobre la que Entity Framework traduce consultas LINQ a SQL Server.

## Func y Action

```csharp
// Func<T, TResult>: Delegado genérico con retorno
Func<Producto, bool> esCaro = p => p.Precio > 100;
var caros = productos.Where(esCaro);

// Func con múltiples parámetros
Func<decimal, decimal, decimal> calcularIVA = (precio, tasa) =>
    precio * (1 + tasa);
var conIVA = productos.Select(p =>
    calcularIVA(p.Precio, 0.16m));

// Action<T>: Delegado sin retorno
Action<Producto> imprimir = p =>
    Console.WriteLine($"{p.Nombre}: ${p.Precio}");
productos.ToList().ForEach(imprimir);
```

## Expression<T> — Árboles de expresión

```csharp
// Expression vs Func
Func<Producto, bool> func = p => p.Precio > 100;     // Código ejecutable
Expression<Func<Producto, bool>> expr = p => p.Precio > 100;  // Árbol de datos

// EF Core usa Expression para traducir a SQL
// Esto se traduce a: WHERE Precio > 100
var resultado = context.Productos.Where(expr).ToList();

// Construcción dinámica de expresiones
var param = Expression.Parameter(typeof(Producto), "p");
var prop = Expression.Property(param, "Precio");
var constant = Expression.Constant(100m);
var comparison = Expression.GreaterThan(prop, constant);
var lambda = Expression.Lambda<Func<Producto, bool>>(comparison, param);

var dinamicos = context.Productos.Where(lambda).ToList();
```

> 💡 **Tip:** Usa `Func<T>` para LINQ to Objects (en memoria) y `Expression<Func<T>>` para LINQ to SQL / EF Core (base de datos). EF Core no puede traducir `Func<T>` a SQL; necesita `Expression<T>` para analizar la consulta.

---

# 12 — Ejecución Diferida vs Inmediata

La ejecución diferida (deferred execution) es uno de los conceptos más importantes de LINQ. Significa que la consulta no se ejecuta en el momento de su definición, sino cuando se itera sobre los resultados. Esto permite construir consultas complejas por etapas sin penalización de rendimiento. La ejecución inmediata ocurre cuando se llama a un operador que materializa los resultados como `ToList()`, `ToArray()`, `Count()`, `First()`, etc. Comprender la diferencia es crucial para evitar bugs sutiles y problemas de rendimiento, especialmente en la Capa de Acceso a Datos.

## Ejecución diferida (Deferred)

```csharp
// La consulta NO se ejecuta aquí
var consulta = productos.Where(p => p.Precio > 100);

// Se ejecuta AQUÍ al iterar
foreach (var p in consulta) {
    Console.WriteLine(p.Nombre);
}

// Si los datos cambian, la consulta refleja los cambios
productos.Add(new Producto { Nombre = "Nuevo", Precio = 200 });
// La próxima iteración de 'consulta' incluirá el nuevo producto
```

## Ejecución inmediata (Immediate)

```csharp
// ToList() fuerza ejecución inmediata
var lista = productos.Where(p => p.Precio > 100).ToList();

// Count() ejecuta inmediatamente
int total = productos.Count(p => p.Activo);

// First() ejecuta inmediatamente
var primero = productos.First(p => p.Precio > 500);

// ToDictionary() ejecuta inmediatamente
var dict = productos.ToDictionary(p => p.Id);
```

> ⚠️ **Precaución:** Si iteras múltiples veces sobre una consulta diferida, la consulta se ejecuta múltiples veces. Usa `ToList()` para materializar si necesitas iterar más de una vez.

---

# 13 — PLINQ (Parallel LINQ)

PLINQ es una implementación paralela de LINQ que puede ejecutar consultas en múltiples núcleos del procesador simultáneamente. Es ideal para operaciones intensivas de CPU sobre grandes colecciones en memoria. Sin embargo, no es adecuado para operaciones de E/S como consultas a bases de datos, donde Entity Framework ya maneja la asincronía. En una arquitectura de 4 capas, PLINQ se usa en la capa de lógica de negocio para procesamiento pesado de datos ya cargados.

## AsParallel / AsOrdered / WithCancellation

```csharp
// AsParallel: Ejecutar en paralelo
var resultado = productos.AsParallel()
    .Where(p => p.Precio > 100)
    .Select(p => new { p.Nombre, p.Precio * 1.16m })
    .ToList();

// AsOrdered: Mantener orden original
var ordenado = productos.AsParallel().AsOrdered()
    .Where(p => p.Stock > 0)
    .Select(p => p.Nombre)
    .ToList();

// WithDegreeOfParallelism: Limitar hilos
var limitado = datos.AsParallel()
    .WithDegreeOfParallelism(4)
    .Where(x => x.Valor > 1000)
    .ToList();

// WithCancellation: Cancelar operación paralela
var cts = new CancellationTokenSource();
var cancelable = datos.AsParallel()
    .WithCancellation(cts.Token)
    .Select(x => Procesar(x));

// ForAll: Ejecutar acción en paralelo sin reunir resultados
productos.AsParallel()
    .Where(p => p.Stock < 10)
    .ForAll(p => EnviarAlertaStock(p));
```

> ⚠️ **Precaución:** PLINQ no siempre es más rápido. Para colecciones pequeñas o operaciones simples, el overhead de paralelización puede ser mayor que el beneficio. Usa `AsParallel()` solo cuando la operación por elemento sea costosa.

---

# 14 — Patrones Avanzados

Los patrones avanzados de LINQ incluyen la creación de operadores personalizados, el patrón PredicateBuilder para construir filtros dinámicos, y la implementación de tablas pivot usando LINQ. Estos patrones son especialmente útiles en la Capa de Lógica de Negocio de una arquitectura de 4 capas, donde las reglas de negocio requieren consultas complejas y dinámicas.

## Operadores personalizados (Extension Methods)

```csharp
public static class LinqExtensions
{
    // Operador personalizado: Paginar
    public static IEnumerable<T> Paginar<T>(
        this IEnumerable<T> source, int pagina, int tamaño) =>
        source.Skip((pagina - 1) * tamaño).Take(tamaño);

    // Operador: Duplicados
    public static IEnumerable<T> Duplicados<T>(
        this IEnumerable<T> source) =>
        source.GroupBy(x => x)
              .Where(g => g.Count() > 1)
              .Select(g => g.Key);

    // Operador: Chunk (dividir en lotes)
    public static IEnumerable<IEnumerable<T>> Chunk<T>(
        this IEnumerable<T> source, int tamaño)
    {
        var lista = source.ToList();
        for (int i = 0; i < lista.Count; i += tamaño)
            yield return lista.Skip(i).Take(tamaño);
    }
}

// Uso
var pagina3 = productos.Paginar(3, 10);
var duplicados = nombres.Duplicados();
var lotes = pedidos.Chunk(50);
```

## PredicateBuilder — Filtros dinámicos

```csharp
public static class PredicateBuilder
{
    public static Expression<Func<T, bool>> True<T>() =>
        f => true;
    public static Expression<Func<T, bool>> False<T>() =>
        f => false;

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

// Uso: Construir filtro dinámicamente
var filtro = PredicateBuilder.True<Producto>();

if (!string.IsNullOrEmpty(categoria))
    filtro = filtro.And(p => p.Categoria == categoria);
if (precioMin.HasValue)
    filtro = filtro.And(p => p.Precio >= precioMin);
if (soloActivos)
    filtro = filtro.And(p => p.Activo);

var resultado = context.Productos.Where(filtro).ToList();
```

## Pivot con LINQ

```csharp
// Tabla pivot: Ventas por categoría y mes
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
        Total = g.Sum(x => x.Total)
    });
```

---

# 15 — Arquitectura de 4 Capas — Conceptos y Diseño

La arquitectura de 4 capas es un patrón de diseño que separa las responsabilidades de una aplicación en cuatro niveles bien definidos, cada uno con una función específica. Esta separación permite que cada capa sea desarrollada, probada y mantenida de forma independiente, lo que resulta en aplicaciones más robustas, escalables y fáciles de mantener. En el contexto de LINQ con SQL Server, cada capa utiliza LINQ de manera diferente: desde las consultas directas en la Capa de Acceso a Datos hasta las transformaciones y validaciones en la Capa de Lógica de Negocio, y el formateo de datos en la Capa de Presentación.

## Las 4 Capas

### 🎨 Capa de Presentación (UI Layer)
Es la interfaz con el usuario. Puede ser una aplicación de consola, Windows Forms, WPF, ASP.NET MVC, o una API Web. Su única responsabilidad es mostrar datos al usuario y capturar sus entradas. **No debe contener lógica de negocio ni acceso directo a datos.** Usa LINQ exclusivamente para formatear y transformar datos para visualización.

### ⚙️ Capa de Lógica de Negocio (BLL - Business Logic Layer)
Contiene las reglas de negocio, validaciones, cálculos y orquestación de operaciones. Es el "cerebro" de la aplicación. Recibe solicitudes de la capa de presentación, aplica las reglas de negocio usando LINQ, y coordina las operaciones con la capa de datos. **Es donde se concentra el uso más intensivo de LINQ** para filtrar, agrupar, calcular y transformar datos según las reglas del negocio.

### 🗄️ Capa de Acceso a Datos (DAL - Data Access Layer)
Encapsula toda la lógica de acceso a la base de datos SQL Server. Implementa el patrón Repository y utiliza Entity Framework Core o LINQ to SQL para interactuar con la base de datos. Su responsabilidad es traducir las solicitudes de la BLL en consultas LINQ que se ejecutan contra SQL Server, y retornar los resultados como entidades o DTOs.

### 📦 Capa de Entidades (Entity Layer)
Contiene las clases que representan las tablas de la base de datos y los DTOs (Data Transfer Objects) que se usan para transferir datos entre capas. Es la capa más simple pero más compartida, ya que todas las demás capas dependen de ella. Las entidades se mapean directamente a las tablas de SQL Server usando Data Annotations o Fluent API.

## Diagrama de Arquitectura

```
┌─────────────────────────────────────────┐
│         CAPA DE PRESENTACIÓN            │
│   (Console / WinForms / Web API)        │
│   → Formateo, paginación, menús        │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│     CAPA DE LÓGICA DE NEGOCIO (BLL)     │
│   (Servicios, Validaciones, Reglas)     │
│   → LINQ: GroupBy, Aggregate, Where    │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│    CAPA DE ACCESO A DATOS (DAL)         │
│   (Repositorios, DbContext, EF Core)    │
│   → LINQ: Query Syntax, Include, Where │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│        CAPA DE ENTIDADES                │
│   (Clases POCO, DTOs, ViewModels)       │
│   → Data Annotations, Mapeo            │
└─────────────────────────────────────────┘
                 │
                 ▼
        ┌──────────────────┐
        │   SQL Server      │
        │   (Tablas, SPs)   │
        └──────────────────┘
```

## Estructura de la Solución en Visual Studio

```
LINQ_4Capas.sln
├── Entidades/
│   ├── Entidades.csproj
│   ├── Producto.cs
│   ├── Categoria.cs
│   ├── Empleado.cs
│   ├── Venta.cs
│   └── DTOs/
│       ├── ProductoDTO.cs
│       └── VentaDTO.cs
├── DAL/
│   ├── DAL.csproj
│   ├── AppDbContext.cs
│   ├── Interfaces/
│   │   └── IRepositorio.cs
│   ├── Repositorio.cs
│   ├── ProductoRepositorio.cs
│   └── VentaRepositorio.cs
├── BLL/
│   ├── BLL.csproj
│   ├── Interfaces/
│   │   └── IProductoServicio.cs
│   ├── ProductoServicio.cs
│   ├── VentaServicio.cs
│   └── ReporteServicio.cs
└── Presentacion/
    ├── Presentacion.csproj
    └── Program.cs
```

## Scripts SQL Server — Creación de tablas

```sql
-- Base de datos
CREATE DATABASE Linq4CapasDB;
GO

USE Linq4CapasDB;
GO

-- Tabla Categorías
CREATE TABLE Categorias (
    Id            INT IDENTITY(1,1) PRIMARY KEY,
    Nombre        NVARCHAR(100) NOT NULL,
    Descripcion   NVARCHAR(500) NULL,
    Activa        BIT NOT NULL DEFAULT 1,
    FechaCreacion DATETIME2 NOT NULL DEFAULT GETDATE()
);

-- Tabla Productos
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

-- Tabla Empleados
CREATE TABLE Empleados (
    Id            INT IDENTITY(1,1) PRIMARY KEY,
    Nombre        NVARCHAR(100) NOT NULL,
    Apellido      NVARCHAR(100) NOT NULL,
    Departamento  NVARCHAR(50) NOT NULL,
    Salario       DECIMAL(18,2) NOT NULL,
    FechaIngreso  DATE NOT NULL,
    Activo        BIT NOT NULL DEFAULT 1
);

-- Tabla Ventas
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
('Pedro', 'Martinez', 'Ventas', 3800.00, '2020-11-05'),
('Laura', 'Sanchez', 'RRHH', 3200.00, '2022-02-14'),
('Diego', 'Ramirez', 'RRHH', 3400.00, '2021-07-22');
```

## Cadena de conexión (appsettings.json)

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=Linq4CapasDB;Trusted_Connection=True;TrustServerCertificate=True;"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information"
    }
  }
}
```

> ⚠️ **Nota:** Para SQL Server Express use `Server=localhost\\SQLEXPRESS`. Para LocalDB use `Server=(localdb)\\mssqllocaldb`. Nunca almacene credenciales en el código fuente; use Azure Key Vault o variables de entorno para producción.

---

# 16 — Capa de Entidades y Acceso a Datos (DAL)

La Capa de Entidades define las clases POCO (Plain Old CLR Objects) que representan las tablas de SQL Server, y la Capa de Acceso a Datos (DAL) encapsula toda la lógica de interacción con la base de datos usando Entity Framework Core. El patrón Repository proporciona una abstracción sobre el DbContext, permitiendo cambiar la implementación de acceso a datos sin afectar la capa de negocio. Cada repositorio utiliza LINQ para construir consultas que se traducen a SQL y se ejecutan en SQL Server.

## Entidades con Data Annotations

```csharp
// ============================================
// Entidades/Producto.cs
// ============================================
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace Entidades
{
    [Table("Productos")]
    public class Producto
    {
        [Key]
        [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
        public int Id { get; set; }

        [Required]
        [StringLength(200)]
        public string Nombre { get; set; } = string.Empty;

        [StringLength(1000)]
        public string? Descripcion { get; set; }

        [Required]
        [Column(TypeName = "decimal(18,2)")]
        public decimal Precio { get; set; }

        [Required]
        public int Stock { get; set; }

        [Required]
        [ForeignKey(nameof(Categoria))]
        public int CategoriaId { get; set; }

        public bool Activo { get; set; } = true;

        public DateTime FechaCreacion { get; set; } = DateTime.Now;

        // Propiedad de navegación
        public virtual Categoria? Categoria { get; set; }
    }
}
```

```csharp
// ============================================
// Entidades/Categoria.cs
// ============================================
namespace Entidades
{
    [Table("Categorias")]
    public class Categoria
    {
        [Key]
        [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
        public int Id { get; set; }

        [Required]
        [StringLength(100)]
        public string Nombre { get; set; } = string.Empty;

        [StringLength(500)]
        public string? Descripcion { get; set; }

        public bool Activa { get; set; } = true;

        public DateTime FechaCreacion { get; set; } = DateTime.Now;

        // Propiedad de navegación inversa
        public virtual ICollection<Producto> Productos { get; set; } = new List<Producto>();
    }
}
```

```csharp
// ============================================
// Entidades/Venta.cs
// ============================================
namespace Entidades
{
    [Table("Ventas")]
    public class Venta
    {
        [Key]
        [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
        public int Id { get; set; }

        [Required]
        public int ProductoId { get; set; }

        [Required]
        public int Cantidad { get; set; }

        [Required]
        [Column(TypeName = "decimal(18,2)")]
        public decimal PrecioUnitario { get; set; }

        public DateTime FechaVenta { get; set; } = DateTime.Now;

        [Required]
        public int VendedorId { get; set; }

        public virtual Producto? Producto { get; set; }
        public virtual Empleado? Vendedor { get; set; }
    }
}
```

## DTOs (Data Transfer Objects)

```csharp
// ============================================
// Entidades/DTOs/ProductoDTO.cs
// ============================================
namespace Entidades.DTOs
{
    public class ProductoDTO
    {
        public int Id { get; set; }
        public string Nombre { get; set; } = string.Empty;
        public decimal Precio { get; set; }
        public int Stock { get; set; }
        public string Categoria { get; set; } = string.Empty;
        public decimal PrecioConIVA => Precio * 1.16m;
        public string Estado => Stock > 20 ? "Disponible" :
                                Stock > 0  ? "Bajo Stock" : "Agotado";
    }

    public class VentaDTO
    {
        public int Id { get; set; }
        public string Producto { get; set; } = string.Empty;
        public int Cantidad { get; set; }
        public decimal PrecioUnitario { get; set; }
        public decimal Total => Cantidad * PrecioUnitario;
        public string Vendedor { get; set; } = string.Empty;
        public DateTime FechaVenta { get; set; }
    }
}
```

## DbContext — Entity Framework Core

```csharp
// ============================================
// DAL/AppDbContext.cs
// ============================================
using Microsoft.EntityFrameworkCore;
using Entidades;

namespace DAL
{
    public class AppDbContext : DbContext
    {
        public AppDbContext(DbContextOptions<AppDbContext> options)
            : base(options) { }

        // DbSets - Representan las tablas de SQL Server
        public DbSet<Producto> Productos { get; set; }
        public DbSet<Categoria> Categorias { get; set; }
        public DbSet<Empleado> Empleados { get; set; }
        public DbSet<Venta> Ventas { get; set; }

        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            // Configuración Fluent API (alternativa a Data Annotations)

            modelBuilder.Entity<Producto>(entity =>
            {
                entity.HasKey(e => e.Id);
                entity.Property(e => e.Precio).HasColumnType("decimal(18,2)");
                entity.Property(e => e.Nombre).HasMaxLength(200).IsRequired();
                entity.HasOne(e => e.Categoria)
                      .WithMany(c => c.Productos)
                      .HasForeignKey(e => e.CategoriaId)
                      .OnDelete(DeleteBehavior.Restrict);
            });

            modelBuilder.Entity<Venta>(entity =>
            {
                entity.HasKey(e => e.Id);
                entity.Property(e => e.PrecioUnitario).HasColumnType("decimal(18,2)");
                entity.HasOne(e => e.Producto)
                      .WithMany()
                      .HasForeignKey(e => e.ProductoId);
                entity.HasOne(e => e.Vendedor)
                      .WithMany()
                      .HasForeignKey(e => e.VendedorId);
            });

            // Seed data
            modelBuilder.Entity<Categoria>().HasData(
                new Categoria { Id = 1, Nombre = "Computo", Descripcion = "Equipos de computo" },
                new Categoria { Id = 2, Nombre = "Accesorio", Descripcion = "Perifericos" },
                new Categoria { Id = 3, Nombre = "Almacenamiento", Descripcion = "Dispositivos de almacenamiento" },
                new Categoria { Id = 4, Nombre = "Impresion", Descripcion = "Impresoras y suministros" }
            );
        }
    }
}
```

## Interfaz y Repositorio Genérico

```csharp
// ============================================
// DAL/Interfaces/IRepositorio.cs
// ============================================
using System.Linq.Expressions;

namespace DAL.Interfaces
{
    public interface IRepositorio<T> where T : class
    {
        // Consultas LINQ diferidas
        IEnumerable<T> ObtenerTodos();
        IEnumerable<T> Obtener(Expression<Func<T, bool>> filtro);
        T? ObtenerPorId(int id);

        // Consultas con LINQ avanzado
        IEnumerable<T> Obtener(
            Expression<Func<T, bool>>? filtro = null,
            Func<IQueryable<T>, IOrderedQueryable<T>>? ordenarPor = null,
            string? propiedadesIncluidas = null,
            int? pagina = null,
            int? tamañoPagina = null);

        // Agregados con LINQ
        int Contar(Expression<Func<T, bool>>? filtro = null);
        decimal Sumar(Expression<Func<T, decimal>> selector,
                      Expression<Func<T, bool>>? filtro = null);
        T? Primero(Expression<Func<T, bool>> filtro);

        // CRUD
        void Agregar(T entidad);
        void Actualizar(T entidad);
        void Eliminar(int id);
        void Eliminar(T entidad);
    }
}
```

```csharp
// ============================================
// DAL/Repositorio.cs
// ============================================
using Microsoft.EntityFrameworkCore;
using DAL.Interfaces;
using System.Linq.Expressions;

namespace DAL
{
    public class Repositorio<T> : IRepositorio<T> where T : class
    {
        protected readonly AppDbContext _context;
        protected readonly DbSet<T> _dbSet;

        public Repositorio(AppDbContext context)
        {
            _context = context;
            _dbSet = context.Set<T>();
        }

        // ── Obtener Todos ──
        public virtual IEnumerable<T> ObtenerTodos()
        {
            return _dbSet.AsNoTracking().ToList();
        }

        // ── Obtener con filtro LINQ ──
        public virtual IEnumerable<T> Obtener(Expression<Func<T, bool>> filtro)
        {
            return _dbSet.Where(filtro).AsNoTracking().ToList();
        }

        // ── Obtener por ID ──
        public virtual T? ObtenerPorId(int id)
        {
            return _dbSet.Find(id);
        }

        // ── Obtener completo con LINQ avanzado ──
        public virtual IEnumerable<T> Obtener(
            Expression<Func<T, bool>>? filtro = null,
            Func<IQueryable<T>, IOrderedQueryable<T>>? ordenarPor = null,
            string? propiedadesIncluidas = null,
            int? pagina = null,
            int? tamañoPagina = null)
        {
            IQueryable<T> consulta = _dbSet;

            // Aplicar filtro Where
            if (filtro != null)
                consulta = consulta.Where(filtro);

            // Incluir propiedades de navegación (Include)
            if (!string.IsNullOrEmpty(propiedadesIncluidas))
            {
                foreach (var prop in propiedadesIncluidas.Split(',',
                    StringSplitOptions.RemoveEmptyEntries))
                    consulta = consulta.Include(prop.Trim());
            }

            // Aplicar ordenamiento (OrderBy / ThenBy)
            if (ordenarPor != null)
                consulta = ordenarPor(consulta);

            // Aplicar paginación (Skip + Take)
            if (pagina.HasValue && tamañoPagina.HasValue)
                consulta = consulta
                    .Skip((pagina.Value - 1) * tamañoPagina.Value)
                    .Take(tamañoPagina.Value);

            return consulta.AsNoTracking().ToList();
        }

        // ── Contar con LINQ ──
        public int Contar(Expression<Func<T, bool>>? filtro = null)
        {
            return filtro == null
                ? _dbSet.Count()
                : _dbSet.Count(filtro);
        }

        // ── Sumar con LINQ ──
        public decimal Sumar(Expression<Func<T, decimal>> selector,
                             Expression<Func<T, bool>>? filtro = null)
        {
            return filtro == null
                ? _dbSet.Sum(selector)
                : _dbSet.Where(filtro).Sum(selector);
        }

        // ── Primero con LINQ ──
        public T? Primero(Expression<Func<T, bool>> filtro)
        {
            return _dbSet.FirstOrDefault(filtro);
        }

        // ── CRUD ──
        public void Agregar(T entidad) => _dbSet.Add(entidad);
        public void Actualizar(T entidad) => _dbSet.Update(entidad);
        public void Eliminar(int id) => _dbSet.Remove(ObtenerPorId(id)!);
        public void Eliminar(T entidad) => _dbSet.Remove(entidad);
    }
}
```

## Repositorio Específico — ProductoRepositorio

```csharp
// ============================================
// DAL/ProductoRepositorio.cs
// ============================================
using Microsoft.EntityFrameworkCore;
using Entidades;

namespace DAL
{
    public class ProductoRepositorio : Repositorio<Producto>
    {
        public ProductoRepositorio(AppDbContext context) : base(context) { }

        // LINQ: Productos con su categoría (Include)
        public IEnumerable<Producto> ObtenerConCategoria()
        {
            return _dbSet
                .Include(p => p.Categoria)
                .AsNoTracking()
                .ToList();
        }

        // LINQ: Productos por categoría con Join
        public IEnumerable<Producto> ObtenerPorCategoria(int categoriaId)
        {
            return _dbSet
                .Where(p => p.CategoriaId == categoriaId && p.Activo)
                .OrderBy(p => p.Nombre)
                .Include(p => p.Categoria)
                .AsNoTracking()
                .ToList();
        }

        // LINQ: Productos con stock bajo
        public IEnumerable<Producto> ObtenerStockBajo(int limiteStock = 10)
        {
            return _dbSet
                .Where(p => p.Stock <= limiteStock && p.Activo)
                .OrderBy(p => p.Stock)
                .Include(p => p.Categoria)
                .AsNoTracking()
                .ToList();
        }

        // LINQ: Búsqueda con Contains
        public IEnumerable<Producto> Buscar(string termino)
        {
            return _dbSet
                .Where(p => p.Nombre.Contains(termino) ||
                            (p.Descripcion != null && p.Descripcion.Contains(termino)))
                .OrderBy(p => p.Nombre)
                .AsNoTracking()
                .ToList();
        }

        // LINQ: Estadísticas por categoría (GroupBy + Aggregates)
        public object EstadisticasPorCategoria()
        {
            return _dbSet
                .Include(p => p.Categoria)
                .Where(p => p.Activo)
                .GroupBy(p => p.Categoria!.Nombre)
                .Select(g => new {
                    Categoria = g.Key,
                    Cantidad = g.Count(),
                    PrecioPromedio = g.Average(p => p.Precio),
                    PrecioMaximo = g.Max(p => p.Precio),
                    StockTotal = g.Sum(p => p.Stock),
                    ValorInventario = g.Sum(p => p.Precio * p.Stock)
                })
                .OrderByDescending(x => x.ValorInventario)
                .ToList();
        }
    }
}
```

---

# 17 — Capa de Lógica de Negocio (BLL)

La Capa de Lógica de Negocio (BLL) es el corazón de la aplicación. Contiene las reglas de negocio, validaciones, cálculos y orquestación de operaciones. Los servicios de la BLL reciben solicitudes de la capa de presentación, aplican las reglas usando LINQ, y coordinan las operaciones con la DAL. Cada servicio encapsula una entidad de negocio y expone métodos que representan las operaciones que el usuario puede realizar. La BLL utiliza LINQ extensivamente para transformar entidades en DTOs, aplicar reglas de validación, calcular métricas y generar reportes.

## Interfaz de Servicio

```csharp
// ============================================
// BLL/Interfaces/IProductoServicio.cs
// ============================================
using Entidades.DTOs;

namespace BLL.Interfaces
{
    public interface IProductoServicio
    {
        // Consultas
        IEnumerable<ProductoDTO> ObtenerTodos();
        ProductoDTO? ObtenerPorId(int id);
        IEnumerable<ProductoDTO> Buscar(string termino);
        IEnumerable<ProductoDTO> ObtenerPorCategoria(int categoriaId);
        IEnumerable<ProductoDTO> ObtenerStockBajo();

        // Reportes con LINQ
        object EstadisticasPorCategoria();
        object TopProductos(int cantidad = 5);
        object VentasPorMes(int año);
        object VentasPorVendedor(DateTime desde, DateTime hasta);

        // CRUD
        int CrearProducto(ProductoDTO dto);
        bool ActualizarProducto(ProductoDTO dto);
        bool EliminarProducto(int id);

        // Paginación
        (IEnumerable<ProductoDTO> Items, int TotalRegistros) ObtenerPaginado(
            int pagina, int tamaño, string? buscar = null, int? categoriaId = null);
    }
}
```

## ProductoServicio — Implementación

```csharp
// ============================================
// BLL/ProductoServicio.cs
// ============================================
using DAL;
using DAL.Interfaces;
using Entidades;
using Entidades.DTOs;
using BLL.Interfaces;
using Microsoft.EntityFrameworkCore;

namespace BLL
{
    public class ProductoServicio : IProductoServicio
    {
        private readonly ProductoRepositorio _productoRepo;
        private readonly IRepositorio<Venta> _ventaRepo;
        private readonly IRepositorio<Categoria> _categoriaRepo;

        public ProductoServicio(
            ProductoRepositorio productoRepo,
            IRepositorio<Venta> ventaRepo,
            IRepositorio<Categoria> categoriaRepo)
        {
            _productoRepo = productoRepo;
            _ventaRepo = ventaRepo;
            _categoriaRepo = categoriaRepo;
        }

        // ── Obtener Todos: Entidad → DTO con LINQ Select ──
        public IEnumerable<ProductoDTO> ObtenerTodos()
        {
            return _productoRepo.ObtenerConCategoria()
                .Select(p => new ProductoDTO
                {
                    Id = p.Id,
                    Nombre = p.Nombre,
                    Precio = p.Precio,
                    Stock = p.Stock,
                    Categoria = p.Categoria?.Nombre ?? "Sin categoría"
                })
                .OrderBy(p => p.Nombre)
                .ToList();
        }

        // ── Obtener Por ID: Select + FirstOrDefault ──
        public ProductoDTO? ObtenerPorId(int id)
        {
            var producto = _productoRepo.ObtenerPorId(id);
            if (producto == null) return null;

            return new ProductoDTO
            {
                Id = producto.Id,
                Nombre = producto.Nombre,
                Precio = producto.Precio,
                Stock = producto.Stock,
                Categoria = producto.Categoria?.Nombre ?? ""
            };
        }

        // ── Buscar: LINQ Where + Contains ──
        public IEnumerable<ProductoDTO> Buscar(string termino)
        {
            if (string.IsNullOrWhiteSpace(termino))
                return ObtenerTodos();

            return _productoRepo.Buscar(termino)
                .Select(p => new ProductoDTO
                {
                    Id = p.Id,
                    Nombre = p.Nombre,
                    Precio = p.Precio,
                    Stock = p.Stock,
                    Categoria = p.Categoria?.Nombre ?? ""
                })
                .ToList();
        }

        // ── Estadísticas: GroupBy + Sum + Average + Max ──
        public object EstadisticasPorCategoria()
        {
            return _productoRepo.EstadisticasPorCategoria();
        }

        // ── Top Productos: OrderByDescending + Take ──
        public object TopProductos(int cantidad = 5)
        {
            return _ventaRepo.Obtener(propiedadesIncluidas: "Producto")
                .GroupBy(v => v.Producto?.Nombre ?? "Desconocido")
                .Select(g => new
                {
                    Producto = g.Key,
                    CantidadVendida = g.Sum(v => v.Cantidad),
                    Ingresos = g.Sum(v => v.Cantidad * v.PrecioUnitario)
                })
                .OrderByDescending(x => x.Ingresos)
                .Take(cantidad)
                .ToList();
        }

        // ── Ventas por Mes: GroupBy + Sum con filtro ──
        public object VentasPorMes(int año)
        {
            return _ventaRepo.Obtener(propiedadesIncluidas: "Producto")
                .Where(v => v.FechaVenta.Year == año)
                .GroupBy(v => v.FechaVenta.Month)
                .Select(g => new
                {
                    Mes = g.Key,
                    TotalVentas = g.Sum(v => v.Cantidad * v.PrecioUnitario),
                    CantidadTransacciones = g.Count(),
                    TicketPromedio = g.Average(v => v.Cantidad * v.PrecioUnitario)
                })
                .OrderBy(x => x.Mes)
                .ToList();
        }

        // ── Ventas por Vendedor: Join + GroupBy ──
        public object VentasPorVendedor(DateTime desde, DateTime hasta)
        {
            return _ventaRepo.Obtener(
                filtro: v => v.FechaVenta >= desde && v.FechaVenta <= hasta,
                propiedadesIncluidas: "Producto,Vendedor")
                .GroupBy(v => v.Vendedor != null
                    ? $"{v.Vendedor.Nombre} {v.Vendedor.Apellido}"
                    : "Desconocido")
                .Select(g => new
                {
                    Vendedor = g.Key,
                    Total = g.Sum(v => v.Cantidad * v.PrecioUnitario),
                    Cantidad = g.Count()
                })
                .OrderByDescending(x => x.Total)
                .ToList();
        }

        // ── Paginación: Skip + Take con filtros dinámicos ──
        public (IEnumerable<ProductoDTO> Items, int TotalRegistros) ObtenerPaginado(
            int pagina, int tamaño, string? buscar = null, int? categoriaId = null)
        {
            var consulta = _productoRepo.ObtenerConCategoria().AsQueryable();

            // Filtro dinámico con LINQ
            if (!string.IsNullOrWhiteSpace(buscar))
                consulta = consulta.Where(p =>
                    p.Nombre.Contains(buscar) ||
                    (p.Descripcion != null && p.Descripcion.Contains(buscar)));

            if (categoriaId.HasValue)
                consulta = consulta.Where(p => p.CategoriaId == categoriaId.Value);

            var total = consulta.Count();

            var items = consulta
                .OrderBy(p => p.Nombre)
                .Skip((pagina - 1) * tamaño)
                .Take(tamaño)
                .Select(p => new ProductoDTO
                {
                    Id = p.Id,
                    Nombre = p.Nombre,
                    Precio = p.Precio,
                    Stock = p.Stock,
                    Categoria = p.Categoria?.Nombre ?? ""
                })
                .ToList();

            return (items, total);
        }

        // ── CRUD ──
        public int CrearProducto(ProductoDTO dto)
        {
            // Validación con LINQ Any
            if (string.IsNullOrWhiteSpace(dto.Nombre))
                throw new ArgumentException("El nombre es obligatorio");

            if (dto.Precio <= 0)
                throw new ArgumentException("El precio debe ser mayor a 0");

            var producto = new Producto
            {
                Nombre = dto.Nombre,
                Precio = dto.Precio,
                Stock = dto.Stock,
                CategoriaId = int.Parse(dto.Categoria),
                Activo = true
            };

            _productoRepo.Agregar(producto);
            _context.SaveChanges();
            return producto.Id;
        }

        public bool ActualizarProducto(ProductoDTO dto)
        {
            var producto = _productoRepo.ObtenerPorId(dto.Id);
            if (producto == null) return false;

            producto.Nombre = dto.Nombre;
            producto.Precio = dto.Precio;
            producto.Stock = dto.Stock;

            _productoRepo.Actualizar(producto);
            _context.SaveChanges();
            return true;
        }

        public bool EliminarProducto(int id)
        {
            var producto = _productoRepo.ObtenerPorId(id);
            if (producto == null) return false;

            // Soft delete: marcar como inactivo
            producto.Activo = false;
            _productoRepo.Actualizar(producto);
            _context.SaveChanges();
            return true;
        }
    }
}
```

## ReporteServicio — Reportes avanzados con LINQ

```csharp
// ============================================
// BLL/ReporteServicio.cs
// ============================================
using DAL;
using Entidades.DTOs;

namespace BLL
{
    public class ReporteServicio
    {
        private readonly AppDbContext _context;

        public ReporteServicio(AppDbContext context)
        {
            _context = context;
        }

        // Reporte: Resumen general del inventario
        public object ResumenInventario()
        {
            return _context.Productos
                .Where(p => p.Activo)
                .GroupBy(p => 1) // Agrupar todo en un solo grupo
                .Select(g => new
                {
                    TotalProductos = g.Count(),
                    ValorTotalInventario = g.Sum(p => p.Precio * p.Stock),
                    PrecioPromedio = g.Average(p => p.Precio),
                    StockTotal = g.Sum(p => p.Stock),
                    ProductosAgotados = g.Count(p => p.Stock == 0),
                    ProductosBajoStock = g.Count(p => p.Stock > 0 && p.Stock <= 10)
                })
                .First();
        }

        // Reporte: Ventas por categoría (Pivot)
        public object VentasPorCategoriaPivot(int año)
        {
            return _context.Ventas
                .Include(v => v.Producto)
                    .ThenInclude(p => p.Categoria)
                .Where(v => v.FechaVenta.Year == año)
                .GroupBy(v => new { v.Producto.Categoria.Nombre, v.FechaVenta.Month })
                .Select(g => new
                {
                    Categoria = g.Key.Nombre,
                    Mes = g.Key.Month,
                    Total = g.Sum(v => v.Cantidad * v.PrecioUnitario)
                })
                .ToList()
                .GroupBy(x => x.Categoria)
                .Select(g => new
                {
                    Categoria = g.Key,
                    Ene = g.FirstOrDefault(x => x.Mes == 1)?.Total ?? 0,
                    Feb = g.FirstOrDefault(x => x.Mes == 2)?.Total ?? 0,
                    Mar = g.FirstOrDefault(x => x.Mes == 3)?.Total ?? 0,
                    Abr = g.FirstOrDefault(x => x.Mes == 4)?.Total ?? 0,
                    May = g.FirstOrDefault(x => x.Mes == 5)?.Total ?? 0,
                    Jun = g.FirstOrDefault(x => x.Mes == 6)?.Total ?? 0,
                    Total = g.Sum(x => x.Total)
                })
                .OrderByDescending(x => x.Total)
                .ToList();
        }

        // Reporte: Tendencia de ventas mensual
        public object TendenciaVentas(int año)
        {
            var meses = Enumerable.Range(1, 12);

            return from m in meses
                   join v in _context.Ventas
                       .Where(v => v.FechaVenta.Year == año)
                       .GroupBy(v => v.FechaVenta.Month)
                       .Select(g => new { Mes = g.Key, Total = g.Sum(v => v.Cantidad * v.PrecioUnitario) })
                       on m equals v.Mes into gv
                   from v in gv.DefaultIfEmpty()
                   select new
                   {
                       Mes = System.Globalization.CultureInfo.CurrentCulture
                           .GetMonthName(m),
                       Total = v?.Total ?? 0
                   };
        }
    }
}
```

---

# 18 — Capa de Presentación (UI)

La Capa de Presentación es la interfaz con el usuario. En este caso, implementamos una aplicación de consola que consume los servicios de la BLL. La capa de presentación usa LINQ principalmente para formatear datos, crear menús interactivos, y mostrar resultados de manera legible. Nunca debe acceder directamente a la DAL ni contener lógica de negocio; su única responsabilidad es presentar la información y capturar las entradas del usuario.

## Program.cs — Aplicación de Consola

```csharp
// ============================================
// Presentacion/Program.cs
// ============================================
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.DependencyInjection;
using DAL;
using DAL.Interfaces;
using BLL;
using BLL.Interfaces;
using Entidades.DTOs;

class Program
{
    static IProductoServicio? _productoServicio;
    static IServiceProvider? _serviceProvider;

    static void Main(string[] args)
    {
        // ── Configurar Inyección de Dependencias ──
        var services = new ServiceCollection();
        services.AddDbContext<AppDbContext>(options =>
            options.UseSqlServer(
                "Server=localhost;Database=Linq4CapasDB;" +
                "Trusted_Connection=True;TrustServerCertificate=True;"));

        services.AddScoped(typeof(IRepositorio<>), typeof(Repositorio<>));
        services.AddScoped<ProductoRepositorio>();
        services.AddScoped<IProductoServicio, ProductoServicio>();
        services.AddScoped<ReporteServicio>();

        _serviceProvider = services.BuildServiceProvider();
        _productoServicio = _serviceProvider
            .GetRequiredService<IProductoServicio>();

        // ── Menú Principal ──
        bool salir = false;
        while (!salir)
        {
            Console.Clear();
            MostrarEncabezado("LINQ 4 Capas - Sistema de Gestión");
            Console.WriteLine("  1. Listar todos los productos");
            Console.WriteLine("  2. Buscar productos");
            Console.WriteLine("  3. Productos por categoría");
            Console.WriteLine("  4. Productos con stock bajo");
            Console.WriteLine("  5. Estadísticas por categoría");
            Console.WriteLine("  6. Top 5 productos vendidos");
            Console.WriteLine("  7. Ventas por mes");
            Console.WriteLine("  8. Paginación de productos");
            Console.WriteLine("  9. Crear producto");
            Console.WriteLine("  10. Actualizar producto");
            Console.WriteLine("  11. Eliminar producto");
            Console.WriteLine("  0. Salir");
            Console.Write("\n  Seleccione una opción: ");

            switch (Console.ReadLine())
            {
                case "1": ListarProductos(); break;
                case "2": BuscarProductos(); break;
                case "3": ProductosPorCategoria(); break;
                case "4": ProductosStockBajo(); break;
                case "5": EstadisticasCategoria(); break;
                case "6": TopProductos(); break;
                case "7": VentasPorMes(); break;
                case "8": Paginacion(); break;
                case "9": CrearProducto(); break;
                case "10": ActualizarProducto(); break;
                case "11": EliminarProducto(); break;
                case "0": salir = true; break;
            }

            if (!salir)
            {
                Console.WriteLine("\nPresione cualquier tecla para continuar...");
                Console.ReadKey();
            }
        }
    }

    // ── Listar Todos: LINQ Select para formateo ──
    static void ListarProductos()
    {
        var productos = _productoServicio!.ObtenerTodos();

        MostrarEncabezado("Todos los Productos");

        // LINQ en la presentación: formatear salida
        var filas = productos.Select((p, i) =>
            $"  {i + 1,3}. {p.Nombre,-25} | {p.Categoria,-15} | ${p.Precio,8:F2} | Stock: {p.Stock,4} | {p.Estado}");

        foreach (var fila in filas)
            Console.WriteLine(fila);

        // Totales con LINQ
        Console.WriteLine($"\n  Total: {productos.Count()} productos");
        Console.WriteLine($"  Valor inventario: ${productos.Sum(p => p.Precio * p.Stock):F2}");
    }

    // ── Buscar: LINQ Where + Contains ──
    static void BuscarProductos()
    {
        Console.Write("  Término de búsqueda: ");
        var termino = Console.ReadLine() ?? "";

        var resultados = _productoServicio!.Buscar(termino);

        MostrarEncabezado($"Resultados para '{termino}'");
        resultados.ToList().ForEach(p =>
            Console.WriteLine($"  {p.Nombre,-25} | ${p.Precio,8:F2} | {p.Estado}"));
    }

    // ── Estadísticas: LINQ GroupBy + Aggregates ──
    static void EstadisticasCategoria()
    {
        var stats = _productoServicio!.EstadisticasPorCategoria();
        MostrarEncabezado("Estadísticas por Categoría");
        Console.WriteLine(stats); // Serializar objeto dinámico
    }

    // ── Paginación: LINQ Skip + Take ──
    static void Paginacion()
    {
        int pagina = 1;
        int tamaño = 5;

        while (true)
        {
            var (items, total) = _productoServicio!
                .ObtenerPaginado(pagina, tamaño);

            int totalPaginas = (int)Math.Ceiling((double)total / tamaño);

            MostrarEncabezado($"Productos - Página {pagina}/{totalPaginas}");

            items.ToList().ForEach(p =>
                Console.WriteLine($"  {p.Nombre,-25} | ${p.Precio,8:F2} | {p.Estado}"));

            Console.WriteLine($"\n  [A]nterior | [S]iguiente | [V]olver");
            var key = Console.ReadKey(true).Key;

            if (key == ConsoleKey.A && pagina > 1) pagina--;
            else if (key == ConsoleKey.S && pagina < totalPaginas) pagina++;
            else if (key == ConsoleKey.V) break;
        }
    }

    static void MostrarEncabezado(string titulo)
    {
        Console.WriteLine($"\n  ╔══════════════════════════════════════════╗");
        Console.WriteLine($"  ║  {titulo,-40}  ║");
        Console.WriteLine($"  ╚══════════════════════════════════════════╝\n");
    }
}
```

---

# 19 — Entity Framework Core con SQL Server

Entity Framework Core es el ORM oficial de Microsoft para .NET y el proveedor LINQ más utilizado para conectarse a SQL Server. Traduce consultas LINQ a código SQL de manera automática, maneja el seguimiento de cambios (change tracking), y proporciona migraciones para evolucionar el esquema de la base de datos. En una arquitectura de 4 capas, EF Core reside exclusivamente en la Capa de Acceso a Datos (DAL), donde el `DbContext` actúa como la unidad de trabajo (Unit of Work) y cada `DbSet` funciona como un repositorio.

## Configuración y Migraciones

```bash
# Instalar herramientas EF Core
dotnet tool install --global dotnet-ef

# Crear la primera migración
dotnet ef migrations add InitialCreate --project DAL --startup-project Presentacion

# Aplicar migración a SQL Server
dotnet ef database update --project DAL --startup-project Presentacion

# Crear migración posterior
dotnet ef migrations add AgregarCampoDescripcion --project DAL --startup-project Presentacion
```

## Consultas LINQ con EF Core

```csharp
// ── Include / ThenInclude: Carga ansiosa (Eager Loading) ──
var productosConCategoria = _context.Productos
    .Include(p => p.Categoria)
    .ToList();

// Include anidado: Producto → Categoria, Ventas → Producto
var ventasDetalle = _context.Ventas
    .Include(v => v.Producto)
        .ThenInclude(p => p.Categoria)
    .Include(v => v.Vendedor)
    .ToList();

// ── AsNoTracking: Consultas de solo lectura (mejor rendimiento) ──
var productos = _context.Productos
    .AsNoTracking()
    .Where(p => p.Activo)
    .OrderBy(p => p.Nombre)
    .ToList();

// ── Select: Proyección (solo campos necesarios → SQL optimizado) ──
var resumen = _context.Productos
    .Where(p => p.Activo)
    .Select(p => new {
        p.Nombre,
        p.Precio,
        Categoria = p.Categoria.Nombre
    })
    .ToList();
// SQL generado: SELECT p.Nombre, p.Precio, c.Nombre FROM Productos p JOIN Categorias c...
```

## Eager Loading vs Lazy Loading

```csharp
// ── Eager Loading: Carga explícita con Include ──
// Se carga todo en una sola consulta SQL con JOIN
var eager = _context.Productos
    .Include(p => p.Categoria)
    .ToList();
// SQL: SELECT * FROM Productos p LEFT JOIN Categorias c ON p.CategoriaId = c.Id

// ── Lazy Loading: Carga automática al acceder a la propiedad ──
// Requiere configuración: services.AddDbContext<...>(o => o.UseLazyLoadingProxies())
var lazy = _context.Productos.First();
var cat = lazy.Categoria; // ← Se ejecuta consulta SQL adicional aquí
// SQL 1: SELECT TOP 1 * FROM Productos
// SQL 2: SELECT * FROM Categorias WHERE Id = @p0

// ── Explicit Loading: Carga manual explícita ──
var producto = _context.Productos.First();
_context.Entry(producto).Reference(p => p.Categoria).Load();
```

> 💡 **Mejor práctica:** Usa `AsNoTracking()` para consultas de solo lectura (hasta 30% más rápido). Usa Eager Loading (`Include`) cuando sabes que necesitas las relaciones. Evita Lazy Loading en APIs web porque genera múltiples consultas SQL (problema N+1).

## Consultas Raw SQL con LINQ

```csharp
// FromSqlRaw: Consulta SQL pura
var productos = _context.Productos
    .FromSqlRaw("SELECT * FROM Productos WHERE Precio > {0}", 100)
    .Where(p => p.Activo)  // LINQ se combina con SQL
    .OrderBy(p => p.Precio)
    .ToList();

// FromSqlInterpolated: SQL con interpolación segura
var minimo = 100m;
var resultado = _context.Productos
    .FromSqlInterpolated($"SELECT * FROM Productos WHERE Precio > {minimo}")
    .Include(p => p.Categoria)
    .ToList();

// ExecuteSqlRaw: Para INSERT, UPDATE, DELETE directos
int filas = _context.Database.ExecuteSqlRaw(
    "UPDATE Productos SET Stock = Stock + {0} WHERE CategoriaId = {1}", 10, 1);
```

## Stored Procedures con LINQ

```csharp
// Ejecutar SP que retorna entidades
var productos = _context.Productos
    .FromSqlRaw("EXEC sp_ObtenerProductosPorCategoria @p0", categoriaId)
    .ToList();

// Ejecutar SP que retorna valores escalares
var total = _context.Database
    .SqlQueryRaw<decimal>("EXEC sp_CalcularTotalVentas @p0, @p1", fechaInicio, fechaFin)
    .First();

// SP con parámetros de salida
var paramsOut = new[]
{
    new SqlParameter("@CategoriaId", categoriaId),
    new SqlParameter("@Total", SqlDbType.Decimal) { Direction = ParameterDirection.Output }
};
_context.Database.ExecuteSqlRaw(
    "EXEC @Total = sp_CalcularTotalPorCategoria @CategoriaId, @Total OUTPUT", paramsOut);
decimal resultado = (decimal)paramsOut[1].Value;
```

---

# 20 — Proyecto Integrado — CRUD Completo con 4 Capas

Este capítulo presenta un proyecto integrado que demuestra el funcionamiento completo de una aplicación con arquitectura de 4 capas conectada a SQL Server. Incluye todas las operaciones CRUD (Create, Read, Update, Delete) distribuidas correctamente entre las capas, con LINQ como lenguaje de consulta unificado en toda la solución.

## Configuración de Inicio (Program.cs)

```csharp
// ============================================
// Presentacion/Program.cs — Configuración completa
// ============================================
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.DependencyInjection;
using DAL;
using DAL.Interfaces;
using BLL;
using BLL.Interfaces;

var services = new ServiceCollection();

// 1. Registrar DbContext con SQL Server
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

// 4. Construir proveedor
var serviceProvider = services.BuildServiceProvider();

// 5. Crear base de datos y aplicar migraciones automáticamente
using (var scope = serviceProvider.CreateScope())
{
    var db = scope.ServiceProvider.GetRequiredService<AppDbContext>();
    db.Database.Migrate(); // Crea la BD si no existe
    Console.WriteLine("✓ Base de datos lista");
}

// 6. Ejecutar aplicación
var productoServicio = serviceProvider.GetRequiredService<IProductoServicio>();
var reporteServicio = serviceProvider.GetRequiredService<ReporteServicio>();

// ── DEMOSTRACIÓN CRUD COMPLETO ──

// CREATE: Agregar producto
Console.WriteLine("\n► CREATE: Agregar producto");
int nuevoId = productoServicio.CrearProducto(new ProductoDTO
{
    Nombre = "Webcam HD 1080p",
    Precio = 65.00m,
    Stock = 30,
    Categoria = "2" // Accesorio
});
Console.WriteLine($"  Producto creado con ID: {nuevoId}");

// READ: Listar todos
Console.WriteLine("\n► READ: Todos los productos");
var todos = productoServicio.ObtenerTodos();
todos.Take(5).ToList().ForEach(p =>
    Console.WriteLine($"  {p.Nombre,-25} | ${p.Precio,8:F2} | {p.Estado}"));

// READ: Buscar
Console.WriteLine("\n► READ: Buscar 'Laptop'");
var laptops = productoServicio.Buscar("Laptop");
laptops.ToList().ForEach(p =>
    Console.WriteLine($"  {p.Nombre,-25} | ${p.Precio,8:F2}"));

// READ: Estadísticas
Console.WriteLine("\n► READ: Estadísticas por categoría");
var stats = productoServicio.EstadisticasPorCategoria();
Console.WriteLine(stats);

// UPDATE: Modificar producto
Console.WriteLine("\n► UPDATE: Modificar producto");
var actualizado = productoServicio.ActualizarProducto(new ProductoDTO
{
    Id = nuevoId,
    Nombre = "Webcam HD 1080p Pro",
    Precio = 79.99m,
    Stock = 25,
    Categoria = "2"
});
Console.WriteLine($"  Producto actualizado: {actualizado}");

// DELETE: Eliminar producto (soft delete)
Console.WriteLine("\n► DELETE: Eliminar producto");
var eliminado = productoServicio.EliminarProducto(nuevoId);
Console.WriteLine($"  Producto eliminado: {eliminado}");

// REPORTES LINQ
Console.WriteLine("\n► REPORTE: Top 5 productos");
var top = productoServicio.TopProductos(5);
Console.WriteLine(top);

Console.WriteLine("\n► REPORTE: Resumen inventario");
var resumen = reporteServicio.ResumenInventario();
Console.WriteLine(resumen);
```

**Salida de consola completa:**
```
✓ Base de datos lista

► CREATE: Agregar producto
  Producto creado con ID: 9

► READ: Todos los productos
  Auriculares Sony           | $120.00    | Disponible
  Impresora Epson            | $280.00    | Disponible
  Laptop HP                  | $1200.00   | Disponible
  Monitor Samsung            | $450.00    | Bajo Stock
  Mouse Logitech             | $35.00     | Disponible

► READ: Buscar 'Laptop'
  Laptop HP                  | $1200.00

► READ: Estadísticas por categoría
  Computo        | 3 prod | Prom: $561.67 | Max: $1200.00 | Stock: 68  | Valor: $18,480.00
  Accesorio      | 3 prod | Prom: $78.33  | Max: $120.00  | Stock: 170 | Valor: $6,050.00
  Almacenamiento | 2 prod | Prom: $65.00  | Max: $75.00   | Stock: 110 | Valor: $6,350.00
  Impresion      | 1 prod | Prom: $280.00 | Max: $280.00  | Stock: 12  | Valor: $3,360.00

► UPDATE: Modificar producto
  Producto actualizado: True

► DELETE: Eliminar producto
  Producto eliminado: True

► REPORTE: Top 5 productos
  Laptop HP          | 45 vendidos | $54,000.00
  Monitor Samsung    | 30 vendidos | $13,500.00
  Teclado Mecanico   | 60 vendidos | $4,800.00
  SSD Kingston       | 80 vendidos | $4,400.00
  Auriculares Sony   | 25 vendidos | $3,000.00
```

## Resumen de LINQ por Capa

| Capa | Operadores LINQ más usados | Propósito |
|------|---------------------------|-----------|
| **Presentación** | `Select`, `OrderBy`, `Take`, `Skip`, `Count`, `Sum` | Formateo, paginación, resúmenes visuales |
| **BLL** | `Where`, `GroupBy`, `Aggregate`, `Any`, `All`, `Select` | Validaciones, reglas de negocio, cálculos |
| **DAL** | `Where`, `Include`, `AsNoTracking`, `FirstOrDefault`, `OrderBy` | Consultas SQL, filtrado, carga de relaciones |
| **Entidades** | Data Annotations, `Expression<T>` | Mapeo, validación de modelos |

---

## 📋 Referencia Rápida de Operadores LINQ

### Filtrado
| Operador | Firma | Descripción |
|----------|-------|-------------|
| `Where` | `Where(Func<T,bool>)` | Filtra por predicado |
| `OfType<T>` | `OfType<T>()` | Filtra por tipo |

### Proyección
| Operador | Firma | Descripción |
|----------|-------|-------------|
| `Select` | `Select(Func<T,R>)` | Transforma elementos |
| `SelectMany` | `SelectMany(Func<T,IEnum<R>>)` | Aplana colecciones |

### Ordenamiento
| Operador | Firma | Descripción |
|----------|-------|-------------|
| `OrderBy` | `OrderBy(Func<T,K>)` | Ordena ascendente |
| `OrderByDescending` | `OrderByDescending(Func<T,K>)` | Ordena descendente |
| `ThenBy` | `ThenBy(Func<T,K>)` | Orden secundario asc |
| `ThenByDescending` | `ThenByDescending(Func<T,K>)` | Orden secundario desc |
| `Reverse` | `Reverse()` | Invierte orden |

### Agrupamiento
| Operador | Firma | Descripción |
|----------|-------|-------------|
| `GroupBy` | `GroupBy(Func<T,K>)` | Agrupa por clave (diferido) |
| `ToLookup` | `ToLookup(Func<T,K>)` | Agrupa por clave (inmediato) |

### Join
| Operador | Firma | Descripción |
|----------|-------|-------------|
| `Join` | `Join(inner, outerKey, innerKey, result)` | Inner join |
| `GroupJoin` | `GroupJoin(inner, outerKey, innerKey, result)` | Group join / Left join |
| `Zip` | `Zip(second, result)` | Combina por posición |

### Agregado
| Operador | Firma | Descripción |
|----------|-------|-------------|
| `Count` | `Count()` / `Count(Func<T,bool>)` | Cuenta elementos |
| `LongCount` | `LongCount()` | Cuenta (long) |
| `Sum` | `Sum(Func<T,N>)` | Suma valores |
| `Min` | `Min(Func<T,N>)` | Valor mínimo |
| `Max` | `Max(Func<T,N>)` | Valor máximo |
| `Average` | `Average(Func<T,N>)` | Promedio |
| `Aggregate` | `Aggregate(seed, func, result)` | Agregado personalizado |

### Elementos
| Operador | Firma | Descripción |
|----------|-------|-------------|
| `First` | `First()` / `First(Func<T,bool>)` | Primer elemento |
| `FirstOrDefault` | `FirstOrDefault()` | Primero o default |
| `Last` | `Last()` | Último elemento |
| `LastOrDefault` | `LastOrDefault()` | Último o default |
| `Single` | `Single()` | Elemento único |
| `SingleOrDefault` | `SingleOrDefault()` | Único o default |

### Cuantificación
| Operador | Firma | Descripción |
|----------|-------|-------------|
| `Any` | `Any()` / `Any(Func<T,bool>)` | ¿Hay alguno? |
| `All` | `All(Func<T,bool>)` | ¿Todos cumplen? |
| `Contains` | `Contains(item)` | ¿Contiene elemento? |

### Partición
| Operador | Firma | Descripción |
|----------|-------|-------------|
| `Take` | `Take(int)` | Primeros N |
| `Skip` | `Skip(int)` | Saltar N |
| `TakeWhile` | `TakeWhile(Func<T,bool>)` | Tomar mientras |
| `SkipWhile` | `SkipWhile(Func<T,bool>)` | Saltar mientras |

### Conjuntos
| Operador | Firma | Descripción |
|----------|-------|-------------|
| `Distinct` | `Distinct()` | Sin duplicados |
| `DistinctBy` | `DistinctBy(Func<T,K>)` | Sin duplicados por clave |
| `Union` | `Union(second)` | Unión sin duplicados |
| `Intersect` | `Intersect(second)` | Intersección |
| `Except` | `Except(second)` | Diferencia |

### Conversión
| Operador | Firma | Descripción |
|----------|-------|-------------|
| `ToArray` | `ToArray()` | Convertir a array |
| `ToList` | `ToList()` | Convertir a lista |
| `ToDictionary` | `ToDictionary(key, value)` | Convertir a diccionario |
| `ToHashSet` | `ToHashSet()` | Convertir a HashSet |
| `AsEnumerable` | `AsEnumerable()` | Bajar a IEnumerable |
| `AsQueryable` | `AsQueryable()` | Subir a IQueryable |
| `Cast<T>` | `Cast<T>()` | Convertir tipos |

### Generación
| Operador | Firma | Descripción |
|----------|-------|-------------|
| `Range` | `Enumerable.Range(start, count)` | Rango de enteros |
| `Repeat` | `Enumerable.Repeat(value, count)` | Repetir valor |
| `Empty` | `Enumerable.Empty<T>()` | Secuencia vacía |

---

## 🛠️ Paquetes NuGet Necesarios

```xml
<!-- DAL.csproj -->
<ItemGroup>
  <PackageReference Include="Microsoft.EntityFrameworkCore" Version="8.0.0" />
  <PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="8.0.0" />
  <PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="8.0.0" />
</ItemGroup>

<!-- Presentacion.csproj -->
<ItemGroup>
  <PackageReference Include="Microsoft.EntityFrameworkCore.Design" Version="8.0.0" />
  <PackageReference Include="Microsoft.Extensions.DependencyInjection" Version="8.0.0" />
  <PackageReference Include="Microsoft.Extensions.Configuration.Json" Version="8.0.0" />
</ItemGroup>
```

---

> 📘 **LINQ en C# — Manual Completo** | .NET 8+ | C# 12 | Visual Studio Community 2022 | SQL Server | Arquitectura 4 Capas
