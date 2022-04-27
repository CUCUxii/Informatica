# Explotación de binarios (by CUCUxii)

## Índice 
- [Que es un binario](#Inicio)





---------------------------------------------------------------------------
## ¿Que es un binario?

Un binario, es lo que resulta de compilar un código en lenguaje C o java. Este código se ha traducido a bytes, que equivalen a lenguaje máquina, instrucciones 
que van directamente al procesador sin un intermediario (como puede ser con lenguajes interpretados como Python) Por tanto si intentas abrir un binario con 
nano o vim te encontrarás algunas cadenas legibles entre un lío de "@" y "." 
   - Como ya hemos dicho, son todo bytes, asi que si hay un "x41" lo traducirá a ascii como una "A", pero si hay un byte que no tenga dicho traduccion a ascii, 
   nano los escribirá como  estos "@" y ".".
   
Aunque ya hayamos perdido la capacidad de leer el código del binario (si no tenemos su codigo.c sin compilar claro), esas insutrcciones al procesador de las que
habíamos habaldo antes si que son legibles como código de **Ensamblador** o lenguaje de bajo nivel. El ensamblador es más abstracto que un código Python, 
y si lo ves por primera vez, puede asustar un poco. Pero al final se trata de insutrcciones muy simples (que pasaré a explicar en otro post)

---------------------------------------------------------------------------
## Conocer el binario

   ### Protecciones:
   Los binarios cuentan con varios tipos de protecciones disponibles, conocer estas es ideal para saber que tipo de ataque podríamos probar
   
   

