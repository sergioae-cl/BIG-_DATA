# BIG_DATA


Tarea 4

Caso de Uso Seleccionado: Catálogo de Productos para E-commerce
¿Por qué MongoDB? Un catálogo de productos es el caso de uso para bases de datos orientadas a documentos. Ademas de guardar toda la información del producto (incluyendo comentarios y especificaciones técnicas) en un solo documento, evitando JOINs costosos.

Diseño del Esquema:

Base de Datos: TechStoreDB
Colección Principal: productos
Jerarquia de la coleccion:
```
{
  "_id": ObjectId("..."),
  "nombre": "Laptop Gaming X1",
  "categoria": "Laptops",
  "precio": 1200.50,
  "stock": 15,
  "activo": true,
  "fecha_lanzamiento": ISODate("2023-01-15T00:00:00Z"),
  "caracteristicas": {  // Objeto anidado
    "marca": "TechBrand",
    "color": "Negro",
    "peso": "2.5kg"
  },
  "etiquetas": ["gaming", "oferta", "alto rendimiento"], // Array
  "valoraciones": [ // Array de objetos (One-to-Few relationship)
    {
      "usuario": "user123",
      "puntaje": 5,
      "comentario": "Excelente máquina"
    }
  ]
}
```

Creacion de la BD

Mediante la conexion ala localhost se crea la base de datos con lo siguientes parametros

<img width="632" height="495" alt="image" src="https://github.com/user-attachments/assets/00a3890e-73dc-4e64-b3f8-172e4e5ab831" />


Añadir la informacion mediante el sigioente escript que generarar masivamente los registros

```
// 1. Usar la base de datos
use TechStoreDB;

// 2. Script para generar 100 productos
var categorias = ["Laptops", "Smartphones", "Accesorios", "Monitores", "Audio"];
var marcas = ["Sony", "Samsung", "Apple", "Dell", "Logitech"];
var etiquetas_lista = ["oferta", "nuevo", "envio_gratis", "gaming", "oficina", "pro"];

var productos_batch = [];

for (var i = 1; i <= 100; i++) {
    // Selección aleatoria de datos
    var cat_random = categorias[Math.floor(Math.random() * categorias.length)];
    var marca_random = marcas[Math.floor(Math.random() * marcas.length)];
    var precio_random = parseFloat((Math.random() * 2000 + 10).toFixed(2));
    var stock_random = Math.floor(Math.random() * 100);
    
    // Generar etiquetas aleatorias (1 o 2 etiquetas)
    var tags = [etiquetas_lista[Math.floor(Math.random() * etiquetas_lista.length)]];
    if(Math.random() > 0.5) tags.push(etiquetas_lista[Math.floor(Math.random() * etiquetas_lista.length)]);

    var doc = {
        nombre: "Producto " + cat_random + " " + i,
        categoria: cat_random,
        precio: precio_random,
        stock: stock_random,
        activo: true,
        fecha_creacion: new Date(),
        caracteristicas: {
            marca: marca_random,
            garantia: "1 año",
            origen: "Importado"
        },
        etiquetas: tags,
        rating_promedio: Math.floor(Math.random() * 5) + 1 // 1 a 5 estrellas
    };
    productos_batch.push(doc);
}

// 3. Insertar masivamente
db.productos.insertMany(productos_batch);

print("¡Éxito! Se han insertado " + db.productos.countDocuments() + " documentos.");

```
Aunque mongo procesosa codigos en formato json, el aterior codigo tambien puede ser ejcutado aunque sea en formato JavaScript.

<img width="921" height="648" alt="image" src="https://github.com/user-attachments/assets/f19d8f36-6131-43c2-8c28-6b852b46df3d" />

<img width="776" height="680" alt="image" src="https://github.com/user-attachments/assets/8e73ee7e-ebef-4d56-a470-4eebc7f43103" />

<img width="330" height="113" alt="image" src="https://github.com/user-attachments/assets/caa82b2f-9bea-4949-a9f4-7a8809f67a7c" />


Una ves con con los datos insertados, podemos realizar consultas u operaciones CRUD.

Inserción

```
db.productos.insertOne({
    nombre: "Auriculares Noise Cancelling",
    categoria: "Audio",
    precio: 299.99,
    stock: 50,
    activo: true,
    caracteristicas: { marca: "Sony", color: "Negro" },
    etiquetas: ["nuevo", "viaje"],
    rating_promedio: 5
});
```
Buscar todos los productos de la categoría "Smartphones".

```
db.productos.find({ categoria: "Smartphones" }).pretty();
```
Actualización
```
db.productos.updateOne(
    { nombre: "Auriculares Noise Cancelling" },
    { 
      $set: { precio: 250.00 }, 
      $inc: { stock: -1 } // Restamos 1 al stock simulando una venta
    }
);
```

Eliminación

```
db.productos.deleteOne({ nombre: "Auriculares Noise Cancelling" });
```

Consultas con Filtros y Operadores

Buscar productos caros (precio mayor a 1000) pero con stock bajo (menos o igual a 10).

```
db.productos.find({
    precio: { $gt: 1000 },
    stock: { $lte: 10 }
});
```
<img width="424" height="552" alt="image" src="https://github.com/user-attachments/assets/a24c773a-cce8-4362-9f9e-c8918f65ba47" />

Búsqueda dentro de Arrays ($in, $all) Buscar productos que tengan la etiqueta "gaming" u "oficina".

```
db.productos.find({
    etiquetas: { $in: ["gaming", "oficina"] }
});
```
<img width="666" height="577" alt="image" src="https://github.com/user-attachments/assets/2d5af7c3-977c-4010-a75e-65882756cad3" />

Filtros en Objetos Anidados Buscar productos cuya marca  sea "Apple".

```
db.productos.find({
    "caracteristicas.marca": "Apple"
});
```
<img width="579" height="636" alt="image" src="https://github.com/user-attachments/assets/73cae255-8762-45f3-bb31-a185c55d218b" />


Proyección Mostrar solo nombre y precio de los productos, ocultando el _id

```
db.productos.find(
    { categoria: "Laptops" },
    { nombre: 1, precio: 1, _id: 0 }
);
```
<img width="471" height="645" alt="image" src="https://github.com/user-attachments/assets/04bb9ed5-bdd5-4fdc-b50e-4f3e811ce412" />


Consultas de Agregación

Queremos saber cuántos productos hay por categoría y cuál es el precio promedio.
```
db.productos.aggregate([
    {
        $group: {
            _id: "$categoria", // Agrupar por el campo categoría
            total_productos: { $sum: 1 }, // Contar documentos
            precio_promedio: { $avg: "$precio" }, // Promediar campo precio
            stock_total: { $sum: "$stock" } // Sumar todo el stock
        }
    },
    {
        $sort: { precio_promedio: -1 } // Ordenar de mayor a menor precio
    }
]);
```

<img width="738" height="618" alt="image" src="https://github.com/user-attachments/assets/c523db85-35a9-4141-a024-3a41ce1682bb" />

Productos "Premium" Filtrar productos caros y formatear la salida.

```
db.productos.aggregate([
    {
        $match: { precio: { $gt: 1500 } } // Paso 1: Filtrar
    },
    {
        $project: { // Paso 2: Dar formato al resultado
            producto: "$nombre",
            precio_con_iva: { $multiply: ["$precio", 1.19] }, // Calcular campo nuevo
            marca: "$caracteristicas.marca"
        }
    }
]);
```

<img width="715" height="522" alt="image" src="https://github.com/user-attachments/assets/0e6191c6-8427-4d58-878a-c648865e8d25" />

Documentación

Se eligió un esquema basado en documentos, A diferencia de un modelo relacional donde las características técnicas requerirían una tabla, MongoDB nos permite usar un objeto embebido, en
este caso seria "caracteristicas".

MongoDB permite entrar en objetos anidados directamente en la consulta, lo cual simplifica el filtrado por atributos específicos del producto. insertMany se utiliza para la carga masiva,
lo cual es mucho más eficiente en rendimiento de red que ejecutar 100 veces insertOne.

Diccionario de Datos

<img width="707" height="437" alt="image" src="https://github.com/user-attachments/assets/88572c38-62c4-4817-9f74-bc5e7f1ed529" />




