---
title: "Evidencia   Ejercicio 3"
description: "trabajo en clase en mongodb, se trabajo en la base de datos de Airbnb sobre consultas basicas, ademas de eso se aprendio algunos operadores"
publishDate: 2026-04-29
isFeatured: true
---

```javascript
// Filtter by range
db.usuarios.find({ edad: { $gte: 18, $lte: 30 } })

// Filtter by multiple conditions
db.usuarios.find({
    $and: [
        { edad: { $gte: 18 } },
        { edad: { $lte: 30 } },
        { ciudad: "Buenos Aires" }
    ]
})

// Filtter by multiple conditions with projection
db.usuarios.find(
    {
        $and: [
            { edad: { $gte: 18 } },
            { edad: { $lte: 30 } },
            { ciudad: "Buenos Aires" }
        ]
    },
    { nombre: 1, edad: 1, ciudad: 1, _id: 0 }
)

// $nin operator - Not in
db.usuarios.find({ ciudad: { $nin: ["Buenos Aires", "Cordoba"] } })

// $exists operator - Field exists
db.usuarios.find({ telefono: { $exists: true } })

// $elemMatch operator - Match array elements
db.usuarios.find({ hobbies: { $elemMatch: { $eq: "futbol", size: 2} } })

// $regex operator - Regular expression
db.usuarios.find({ nombre: { $regex: /^J/ } }) // Nombres que comienzan con J


use ("sample_airbnb");


db.listingsAndReviews.find({
    $and: [
    { "address.market": "Porto" }
    ]
});

use ("sample_airbnb");

db.listingsAndReviews.find(
    { "address.market": "Porto" },
    { "name": 1, "price": 1, "address.market": 1, "_id": 0 },// Projection
    { sort: { price: -1 }, // Sort by price descending
      limit: 5 } // Limit to 5 results
);

// Buscar todos los alojamientos tipo apartamento y con amenidades con wifi y television por cable
use ("sample_airbnb");
// $elemMatch operator - Match array elements
db.listingsAndReviews.find(
    { 
        "property_type": "Apartment",
        "amenities": { $elemMatch: { $in: ["Cable TV", "Wifi"] } } 
    },
    { 
        "name": 1, 
        "amenities": 1, 
        "address.country": 1, 
        "_id": 0 
    }
);

// $all operator - Match all elements in an array
use ("sample_airbnb");

db.listingsAndReviews.find(
    { 
        "property_type": "Apartment",
        "amenities": { $all: ["Cable TV", "Wifi"] } 
    }, 
    {
        "name": 0, 
        "amenities": 0, 
        "address.country": 1,
        "beds": 1,
        "_id": 0 
    }
);


db.listingsAndReviews.find(
    {   
        "property_type": "Apartment",
        "amenities": { $all: ["Cable TV", "Wifi"] } 
    }, 
    {
        "name": 0, 
        "amenities": 0, 
        "address.country": 1,
        "beds": 1,
        "_id": 0 
    }
);

