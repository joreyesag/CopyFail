# Reporte Técnico: Mitigación Estructural de CVE-2026-31431 (Copy Fail)

Este laboratorio se enfocó en el análisis de bajo nivel y parche estructural del subsistema criptográfico del Kernel de Linux, específicamente en el módulo `algif_aead` (interfaz de sockets `AF_ALG` para `authencesn`). 

El error original permitía que una operación de descifrado maliciosa forzara una desalineación dentro de las estructuras de listas de dispersión (`scatterlist`). Debido a una optimización 'in-place', el núcleo procesaba el fallo lógico sobre el mismo búfer de recepción (`rsgl_src`). Esto causaba la corrupción permanente en la `page cache` de la memoria RAM del sistema operativo, permitiendo alterar binarios críticos con privilegios elevados como `/usr/bin/su`.

Al aplicar el parche permanente, se modificó la lógica en la función del núcleo introduciendo un operador condicional que evalúa la dirección de la operación. Si la petición es un descifrado, se separa físicamente el canal utilizando el scatterlist de transmisión (`tsgl_src`). Cualquier corrupción secundaria queda aislada y no compromete el binario o las llamadas protegidas de `setuid`, mitigando por completo la escalada a root.
