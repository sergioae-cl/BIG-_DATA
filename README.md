# BIG_DATA


Tarea 4

El presente es una guía interactiva de autoría propia con el fin de ilustrar el proceso de creacion de una Base de datos en Mongo para Iustrar un problema practico y mediante cosultas resolver el problema, este guia es de autoria propia autoría.


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




