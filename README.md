# EXAMEN-03-PH-THREE-PROYECTO-GRUPAL
## PH-Three
PH-Tree es un índice ordenado en un espacio n-dimensional (quad-/oct-/2^n-tree) donde cada dimensión está (de forma predeterminada) indexada por un número entero de 64 bits. El orden del índice sigue el orden z/morton. La implementación predeterminada es efectivamente un 'mapa', es decir, cada clave está asociada con un valor como máximo. Las claves son puntos o cajas en el espacio n-dimensional.

Dos puntos fuertes de PH-Trees son las operaciones rápidas de inserción/eliminación y la escalabilidad con grandes conjuntos de datos. También proporciona consultas de ventana rápidas y consultas de vecinos más cercanos, y se escala bien con dimensiones más altas. La implementación predeterminada está limitada a 63 dimensiones.
![image](https://user-images.githubusercontent.com/78099176/208322132-c7b50d35-4c69-4f6c-97cf-9ed39e07a5ee.png)
