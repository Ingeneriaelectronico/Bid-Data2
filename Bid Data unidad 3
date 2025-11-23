// ==========================================
// PROYECTO: Catálogo de E-commerce NoS
// FECHA: 2025
// ==========================================

// 1. CONEXIÓN Y LIMPIEZA
// Conectamos a la base de datos (se creará si no existe al insertar datos)
var db = connect("mongodb://localhost:27017/ecommerce_db");

// Limpiamos la colección para empezar desde cero en cada ejecución
db.products.drop();
print("✅ Base de datos limpiada. Colección 'products' eliminada.");

// ==========================================
// 2. GENERACIÓN DE DATOS (SEEDING)
// ==========================================

const categories = ["Electronics", "Clothing", "Home", "Sports", "Books"];
const adjectives = ["Super", "Ultra", "Mega", "Eco", "Pro", "Smart", "Casual"];
const nouns = ["Widget", "Device", "Shirt", "Table", "Runner", "Phone", "Lamp"];

let bulkData = [];

// Generamos 120 productos para asegurar +100 documentos
for (let i = 0; i < 120; i++) {
    let category = categories[Math.floor(Math.random() * categories.length)];
    let basePrice = Math.floor(Math.random() * 500) + 10;
    
    // Creamos especificaciones dinámicas según la categoría (Polimorfismo de datos)
    let specs = {};
    if (category === "Electronics") {
        specs = { 
            warranty: "2 years", 
            voltage: "220v", 
            weight_gr: Math.floor(Math.random() * 2000) 
        };
    } else if (category === "Clothing") {
        specs = { 
            material: Math.random() > 0.5 ? "Cotton" : "Polyester", 
            size: ["S", "M", "L"][Math.floor(Math.random() * 3)] 
        };
    } else {
        specs = { imported: Math.random() > 0.5 };
    }

    // Generamos reviews aleatorias
    let reviews = [];
    let numReviews = Math.floor(Math.random() * 5); // 0 a 4 reviews
    for(let j=0; j<numReviews; j++){
        reviews.push({
            user: "user_" + Math.floor(Math.random() * 1000),
            rating: Math.floor(Math.random() * 5) + 1, // 1 a 5 estrellas
            date: new Date()
        });
    }

    let doc = {
        name: `${adjectives[Math.floor(Math.random() * adjectives.length)]} ${nouns[Math.floor(Math.random() * nouns.length)]} ${i}`,
        category: category,
        price: basePrice,
        stock: Math.floor(Math.random() * 100),
        specs: specs,
        tags: ["new", category.toLowerCase(), "sale"],
        reviews: reviews,
        isActive: true,
        createdAt: new Date()
    };
    
    bulkData.push(doc);
}

// Inserción masiva
db.products.insertMany(bulkData);
print(`✅ Se han insertado ${bulkData.length} documentos de prueba.`);


// ==========================================
// 3. CONSULTAS BÁSICAS (CRUD)
// ==========================================
print("\n--- 3.1 LECTURA (READ) ---");
// Buscar 1 producto cualquiera
let product = db.products.findOne();
print("Producto encontrado: " + product.name);

print("\n--- 3.2 ACTUALIZACIÓN (UPDATE) ---");
// Actualizar el precio del producto encontrado y añadir un tag
db.products.updateOne(
    { _id: product._id },
    { 
        $set: { price: 999.99 },
        $push: { tags: "bestseller" }
    }
);
print(`Precio actualizado para ${product.name} a 999.99`);

print("\n--- 3.3 ELIMINACIÓN (DELETE) ---");
// Borrar productos sin stock
let deleteResult = db.products.deleteMany({ stock: 0 });
print(`Productos eliminados (stock 0): ${deleteResult.deletedCount}`);


// ==========================================
// 4. CONSULTAS CON FILTROS Y OPERADORES
// ==========================================
print("\n--- 4.1 FILTROS AVANZADOS ---");

// Consulta: Productos de 'Electronics' con precio mayor a 100 Y stock menor a 50
let expensiveElec = db.products.find({
    category: "Electronics",
    price: { $gt: 100 },
    stock: { $lt: 50 }
}).count();
print(`Electrónica cara y con poco stock (Count): ${expensiveElec}`);

// Consulta: Buscar dentro de objetos anidados (specs)
// Productos de Ropa que sean de material "Cotton"
let cottonClothes = db.products.find({
    category: "Clothing",
    "specs.material": "Cotton"
}).count();
print(`Ropa hecha de Algodón: ${cottonClothes}`);

// Consulta: Buscar productos que tengan al menos una review de 5 estrellas
// Uso de $elemMatch para buscar dentro de arrays de objetos
let topRated = db.products.find({
    reviews: { $elemMatch: { rating: 5 } }
}).count();
print(`Productos con al menos una calificación de 5 estrellas: ${topRated}`);


// ==========================================
// 5. AGREGACIÓN (ANALYTICS)
// ==========================================
print("\n--- 5. AGREGACIONES (PIPELINES) ---");

// AGREGACIÓN 1: Precio promedio por Categoría
// Pipeline:
// 1. $match: Filtramos solo productos activos
// 2. $group: Agrupamos por categoría y calculamos promedio de precio y total de inventario
// 3. $sort: Ordenamos por precio promedio descendente
let avgPriceReport = db.products.aggregate([
    { $match: { isActive: true } },
    { $group: {
        _id: "$category",
        avgPrice: { $avg: "$price" },
        totalStock: { $sum: "$stock" },
        count: { $sum: 1 }
    }},
    { $sort: { avgPrice: -1 } }
]);

print("Reporte: Precio Promedio por Categoría:");
avgPriceReport.forEach(doc => {
    print(`- ${doc._id}: $${doc.avgPrice.toFixed(2)} (Stock total: ${doc.totalStock})`);
});

// AGREGACIÓN 2: Unwind y métricas de reseñas
// Descomponemos el array de reviews para calcular el promedio real de estrellas de toda la tienda
let ratingsReport = db.products.aggregate([
    { $unwind: "$reviews" }, // "Aplana" el array, crea un doc por cada review
    { $group: {
        _id: null, // Agrupar todo junto
        avgRatingGlobal: { $avg: "$reviews.rating" },
        totalReviews: { $sum: 1 }
    }}
]);

if (ratingsReport.hasNext()) {
    let result = ratingsReport.next();
    print(`\nEstadísticas Globales de Satisfacción:`);
    print(`Promedio de Estrellas en la tienda: ${result.avgRatingGlobal.toFixed(2)} / 5.0`);
    print(`Total de reseñas procesadas: ${result.totalReviews}`);
}

print("\n✅ Ejecución finalizada exitosamente.");
