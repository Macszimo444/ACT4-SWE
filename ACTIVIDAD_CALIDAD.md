# Actividad de calidad de software

## Integrantes

- Nombre: Máximo Aguilar González

## Repositorio

- Repositorio: https://github.com/Macszimo444/ACT4-SWE
- Rama: `actividad-calidad`
- Último commit: Pendiente

## Parte 2 — Unit testing

### Pruebas agregadas

Se agregaron ocho pruebas para ampliar la cobertura sin repetir casos que verificaran exactamente el mismo comportamiento:

- Cero minutos como un caso normal de estancia gratuita.
- Dieciséis minutos como el primer valor que debe costar $20.
- Sesenta minutos como el último valor del rango con tarifa de $20.
- Ciento veinte minutos como el final de la primera hora adicional iniciada.
- Ciento veintiún minutos como el inicio de una segunda hora adicional.
- Doscientos cuarenta minutos como el último punto antes de llegar al límite máximo.
- Doscientos cuarenta y un minutos como el punto en que la tarifa alcanza $80.
- Un boleto perdido durante una estancia larga, para comprobar que la condición especial prevalece sobre la tarifa normal.

### P1

Probar muchos valores de una misma región no necesariamente mejora mucho una suite porque todos pueden recorrer las mismas decisiones del programa y producir el mismo tipo de resultado. Por ejemplo, probar 20, 30 y 40 minutos confirma varias veces el rango de $20, pero aporta menos información que probar los valores ubicados a ambos lados de una frontera. Conviene seleccionar casos representativos que ejerciten reglas y caminos diferentes.

### P2

Una frontera importante está entre 15 y 16 minutos. A los 15 minutos el estacionamiento todavía es gratuito, mientras que a los 16 comienza el cobro de $20. Probar ambos valores permite detectar errores como usar `<` en lugar de `<=`.

Otra frontera se encuentra entre 60 y 61 minutos. A los 60 minutos se mantiene la tarifa de $20 y a los 61 debe cobrarse la primera hora adicional iniciada, para un total de $35. Los valores cercanos permiten comprobar que el cambio de regla sucede exactamente en el minuto correcto.

También se comprobó la frontera entre 240 y 241 minutos: la tarifa pasa de $65 a $80 y, desde ese momento, no debe superar el límite máximo.

### P3

No. Que todas las pruebas estén en verde solamente demuestra que el programa cumple los casos y expectativas incluidos en esa suite. Podrían existir entradas no consideradas, una interpretación equivocada de las reglas o defectos en comportamientos que nunca fueron probados. Las pruebas aumentan la confianza en el programa, pero no constituyen una demostración absoluta de que sea correcto en todos los escenarios.

## Code review manual

Esta revisión se realizó antes de consultar SonarQube for IDE.

| Archivo | Hallazgo | Tipo | Severidad | Propuesta |
|---------|----------|------|-----------|-----------|
| `ParkingFeeCalculator.java` | Las cantidades 15, 20, 60, 80 y 150 aparecen directamente en la lógica, por lo que su significado depende de conocer las reglas del negocio. | Maintainability | Media | Declarar constantes con nombres que indiquen el significado de cada límite y tarifa. |

| `ParkingFeeCalculator.java` | El cálculo de horas adicionales convierte la operación a decimal y después aplica `Math.ceil`; funciona, pero obliga a entender detalles de redondeo y conversión de tipos. | Readability | Baja | Expresar el redondeo hacia arriba mediante una fórmula entera o encapsularlo en un método con un nombre descriptivo. |

| `ParkingFeeCalculator.java` | La condición de boleto perdido se evalúa antes que los minutos negativos, por lo que `calculateFee(-1, true)` devuelve $150 aunque la regla indica que los minutos negativos deben rechazarse. | Bug | Media | Aclarar la prioridad de ambas reglas y, si todo valor negativo es inválido, validar los minutos antes de procesar el boleto perdido. |

| `ParkingFeeCalculatorTest.java` | Cada prueba crea nuevamente un `ParkingFeeCalculator`, lo que produce repetición aunque las pruebas sí permanecen independientes. | Testing | Baja | Considerar un atributo inicializado para cada prueba o un método de preparación si la creación del objeto se vuelve más compleja. |

| `LegacyParkingReceipt.java` | La expresión `plate == ""` compara referencias y no el contenido de la cadena; una cadena vacía creada como un objeto diferente puede no ser rechazada. | Bug | Alta | Usar `plate.isEmpty()` después de comprobar que `plate` no sea `null`. |

| `LegacyParkingReceipt.java` | El método imprime directamente en la consola al construir el recibo, introduciendo un efecto secundario que no corresponde al valor que devuelve. | Design | Media | Eliminar la impresión o delegar el registro de eventos a otra responsabilidad. |

| `LegacyParkingReceipt.java` | La expresión ternaria que asigna `free` y la comparación `free == true` son redundantes y dificultan la lectura. | Readability | Baja | Asignar `boolean free = fee == 0;` y evaluar directamente `if (free)`. |

| `LegacyParkingReceipt.java` | Todas las entradas inválidas producen el mismo texto `ERROR`, por lo que quien usa el método no puede distinguir la causa del problema. | Design | Media | Definir una estrategia de validación consistente, como excepciones con mensajes específicos o resultados de error diferenciados. |

## Análisis con SonarQube for IDE

El análisis local no mostró problemas de SonarQube en `ParkingFeeCalculator.java` ni en `ParkingFeeCalculatorTest.java`. Los siguientes hallazgos sí fueron mostrados por la extensión:

| Archivo/línea | Regla o mensaje de Sonar | Explicación con mis palabras | ¿Estoy de acuerdo? |
|---------------|---------------------------|------------------------------|---------------------|
| `LegacyParkingReceipt.java:14` | Strings and Boxed types should be compared using `equals()` (`java:S4973`). | `plate == ""` compara si las dos referencias apuntan al mismo objeto, no si la placa tiene el mismo contenido que una cadena vacía. Por eso podría aceptar algunas cadenas vacías que deberían producir `ERROR`. | Sí. Después de validar que `plate` no sea `null`, se debería usar `plate.isEmpty()` o una comparación equivalente por contenido. |

| `LegacyParkingReceipt.java:26` | Standard outputs should not be used directly to log anything (`java:S106`). | El método usa `System.out.println` como registro al crear el recibo. Esto mezcla la construcción del texto con la escritura directa en la consola y dificulta controlar o desactivar ese mensaje. | Sí. En una aplicación real convendría usar un logger; en este método pequeño también se podría eliminar la impresión si no es necesaria. |

| `LegacyParkingReceipt.java:28,30` | Boolean literals should not be redundant (`java:S1125`). | El código convierte una condición que ya es booleana mediante `? true : false` y después compara la variable con `true`. Ambas operaciones agregan texto sin cambiar el resultado. | Sí. Se puede escribir `boolean free = fee == 0;` y después usar directamente `if (free)`. |

| `UsernamePolicy.java:3` | Utility classes should not have public constructors (`java:S1118`). | La clase sólo ofrece un método estático y no mantiene estado. El constructor público que Java genera automáticamente permite crear objetos que no tienen ninguna utilidad. | Sí. Un constructor privado dejaría claro que la clase se usa únicamente mediante sus métodos estáticos. |

## Code review vs. SonarQube

### P4

Sonar detectó en `UsernamePolicy` que una clase de utilidad no debería conservar el constructor público implícito. Ese punto no apareció en mi revisión manual porque la revisión solicitada se concentró en `ParkingFeeCalculator`, sus pruebas y la clase legacy. La herramienta permitió encontrar un detalle adicional al analizar otro archivo del proyecto.

### P5

Durante la revisión manual observé que `ParkingFeeCalculator` comprueba el boleto perdido antes de validar los minutos negativos. Como consecuencia, `calculateFee(-1, true)` devuelve $150, aunque otra regla indica que un número negativo debe producir `IllegalArgumentException`. Sonar no reportó este punto porque depende de interpretar la prioridad y posible contradicción entre dos reglas del negocio.

### P6

No todos los hallazgos tienen la misma importancia. La comparación `plate == ""` puede ocasionar un comportamiento incorrecto porque algunas cadenas vacías no serían rechazadas. En cambio, eliminar `? true : false` mejora principalmente la legibilidad y no cambia el resultado. Por eso el primer hallazgo merece mayor prioridad que el segundo.

### P7

No. Sonar puede analizar la estructura del código y reconocer patrones posiblemente defectuosos, pero no conoce por sí solo las decisiones del negocio. Necesita una especificación externa para saber que la tarifa correcta entre 16 y 60 minutos es $20. Incluso si el código estuviera bien escrito, Sonar no podría determinar si ese importe debería ser $20, $25 o cualquier otra cantidad.

### P8

El análisis estático no sustituye las pruebas unitarias porque no ejecuta las reglas con entradas concretas para comprobar sus resultados. Las pruebas permiten verificar fronteras, excepciones y combinaciones como 15 frente a 16 minutos o una estancia larga con boleto perdido.

Tampoco sustituye el code review humano porque una persona puede interpretar el contexto, las reglas del negocio, las prioridades y la facilidad de modificar el diseño. Sonar encuentra patrones técnicos de manera consistente, pero no puede decidir por sí solo si el comportamiento implementado corresponde a lo que necesita el estacionamiento. Las tres prácticas se complementan: las pruebas comprueban comportamientos, Sonar detecta patrones y la revisión humana evalúa intención y contexto.
