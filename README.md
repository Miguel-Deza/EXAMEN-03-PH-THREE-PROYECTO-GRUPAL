# EXAMEN-03-PH-THREE-PROYECTO-GRUPAL
## PH-Three
PH-Tree es un índice ordenado en un espacio n-dimensional (quad-/oct-/2^n-tree) donde cada dimensión está (de forma predeterminada) indexada por un número entero de 64 bits. El orden del índice sigue el orden z/morton. La implementación predeterminada es efectivamente un 'mapa', es decir, cada clave está asociada con un valor como máximo. Las claves son puntos o cajas en el espacio n-dimensional.

Dos puntos fuertes de PH-Trees son las operaciones rápidas de inserción/eliminación y la escalabilidad con grandes conjuntos de datos. También proporciona consultas de ventana rápidas y consultas de vecinos más cercanos, y se escala bien con dimensiones más altas. La implementación predeterminada está limitada a 63 dimensiones.
![image](https://user-images.githubusercontent.com/78099176/208322132-c7b50d35-4c69-4f6c-97cf-9ed39e07a5ee.png)


#### Tipos clave

El **PH-Tree Map** admite cinco tipos:

- El `PhTreeD` utiliza claves `PhPointD`, que son vectores/puntos de `double` de 64 bits.
- PhTreeF" utiliza claves "PhPointF", que son vectores/puntos de "float" de 32 bits.
- PhTreeBoxD" utiliza claves "PhBoxD", que consisten en dos "PhPointD" que definen un rectángulo/caja alineado con el eje.
- PhTreeBoxF" utiliza claves "PhBoxF", que consisten en dos "PhPointF" que definen un rectángulo/caja alineado con el eje.
- phTree` utiliza claves `PhPoint`, que son vectores/puntos de `std::int64`.

El **PH-Tree MultiMap** admite tres tipos:

- El `PhTreeMultiMapD` utiliza claves `PhPointD`, que son vectores/puntos de `double` de 64 bits.
- El `PhTreeMultiMapBoxD` utiliza claves `PhBoxD`, que consisten en dos `PhPointD` que definen un rectángulo/caja alineado con el eje.
- PhTreeMultiMap` utiliza claves `PhPoint`, que son vectores/puntos de `std::int64`.

Los tipos de árbol adicionales pueden definirse fácilmente de forma análoga a los tipos anteriores; consulte la declaración de los tipos de árbol
para ver un ejemplo. El soporte para clases clave personalizadas (puntos y cajas) así como mapeados de coordenadas personalizados pueden ser
mediante clases `Converter` personalizadas, véase más abajo. El `PhTreeMultiMap` está por defecto respaldado
por `std::unordered_set` pero esto se puede cambiar a través de un parámetro de plantilla.

Los tipos `PhTree` y `PhTreeMultiMap` están disponibles en `phtree.h` y `phtree_multimap.h`.

<a id="basic-operations"></a>

#### Operaciones básicas

```C++
class MyData { ... };
MyData my_data; 

// Create a 3D point tree with floating point coordinates and a value type of `MyData`.
auto tree = PhTreeD<3, MyData>();

// Create coordinate
PhPointD<3> p{1.1, 1.0, 10.};

// Some operations
tree.emplace(p, my_data);
tree.emplace_hint(hint, p, my_data);
tree.insert(p, my_data);
tree[p] = my_data;
tree.count(p);
tree.find(p);
tree.erase(p);
tree.erase(iterator);
tree.size();
tree.empty();
tree.clear();

// Multi-map only
tree.relocate(p_old, p_new, value);
tree.estimate_count(query);
```

<a id="queries"></a>

#### Consultas

* For-each sobre todos los elementos: `tree.fore_each(callback);`
* Iterador sobre todos los elementos: `auto iterator = tree.begin();`
* For-each con consultas de ventana en forma de caja: `tree.fore_each(PhBoxD(min, max), callback);`
* Iterador para consultas de ventana en forma de caja: `auto q = tree.begin_query(PhBoxD(min, max));`
* Iterador para consultas de _k_ vecinos más cercanos: `auto q = tree.begin_knn_query(k, center_point, distance_function);`
* Formas de consulta personalizadas, como esferas: `tree.for_each(callback, FilterSphere(center, radius, tree.converter()));`

<a id="for-each-example"></a>

##### Por ejemplo

```C++
// Callback for counting entries
struct Counter {
    void operator()(PhPointD<3> key, T& t) {
        ++n_;
    }
    size_t n_ = 0;
};

// Count entries inside of an axis aligned box defined by the two points (1,1,1) and (3,3,3)
Counter callback;
tree.for_each({{1, 1, 1}, {3, 3, 3}}, callback);
// callback.n_ is now the number of entries in the box.
```

<a id="iterator-examples"></a>

##### Ejemplos de iteradores

```C++
// Iterate over all entries
for (auto it : tree) {
    ...
}

// Iterate over all entries inside of an axis aligned box defined by the two points (1,1,1) and (3,3,3)    
for (auto it = tree.begin_query({{1, 1, 1}, {3, 3, 3}}); it != tree.end(); ++it) {
    ...
}

// Find 5 nearest neighbors of (1,1,1)    
for (auto it = tree.begin_knn_query(5, {1, 1, 1}); it != tree.end(); ++it) {
    ...
}
```

<a id="Filters"></a>

##### Filtros

Todas las consultas permiten especificar un filtro adicional. El filtro se aplica a cada par clave/valor que normalmente se devolvería
(sujeto a las restricciones de la consulta) y a cada nodo del árbol que la consulta decida recorrer (también sujeto a las restricciones de la consulta).
a las restricciones de la consulta). Devolver `true` en el filtro no cambia el comportamiento de la consulta, devolver `false` significa que el
significa que el valor actual o el nodo hijo no se devuelve ni se recorre. Puede encontrar un ejemplo de filtro geométrico
en `phtree/common/filter.h` en `FilterAABB`.

```C++
template <dimension_t DIM, typename T>
struct FilterByValueId {
    [[nodiscard]] constexpr bool IsEntryValid(const PhPoint<DIM>& key, const T& value) const {
        // Arbitrary example: Only allow values with even values of id_
        return value.id_ % 2 == 0;
    }
    [[nodiscard]] constexpr bool IsNodeValid(const PhPoint<DIM>& prefix, int bits_to_ignore) const {
        // Allow all nodes
        return true;
    }
};

// Iterate over all entries inside of an axis aligned box defined by the two points (1,1,1) and (3,3,3).
// Return only entries that suffice the filter condition.    
for (auto it = tree.begin_query({1, 1, 1}, {3, 3, 3}, FilterByValueId<3, T>())); it != tree.end(); ++it) {
    ...
}
```

<a id="distance-functions"></a>

##### Función de distancia

Las consultas al vecino más próximo también pueden utilizar métricas de distancia personalizadas, como la distancia L1. Tenga en cuenta que devuelve un iterador
que proporciona una función para obtener la distancia de la entrada actual:

```C++
#include "phtree/phtree.h"

// Find 5 nearest neighbors of (1,1,1) using L1 distance    
for (auto it = tree.begin_knn_query(5, {1, 1, 1}, DistanceL1<3>())); it != tree.end(); ++it) {
    std::cout << "distance = " << it.distance() << std::endl;
    ...
}

```

<a id="converters"></a>