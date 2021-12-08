+++
draft = false
date = 2021-12-08T12:37:40+01:00
title = "Prototype Pollution"
description = ""
slug = ""
authors = ["LeVarez"]
tags = []
categories = []
externalLink = ""
series = []
+++

Primero de todo, qué és la contaminación prototype? Como puede comprometer la seguridad de nuestro sistema?

Este tipo de ataque se lleva a cavo en el lenguage de programación JavaScript, ya que este mismo és prototype-based. Todos los nuevos objetos que se crean llevan las propiedades y mètodos del objeto prototype, el cual contiene funcionalidades bàsicas como ```toString()```, ```constructor()``` y ```hasOwnProperty()```.

Los atacantes puedes hacer uso de esta característica del lenguaje para poder hacer canvios a todos los objetos de la aplicación haciendo una modificación al objeto prtototype. De aquí el nombre de prtotype pollution.

Este objeto puede ser accedido desde cualquier otro objeto a través del atributo ```__proto__```. Y una vez este objeto es modificado todos los objetos de l'aplicación en marcha seràn modificados, tanto los que se creen de nuevo como los que ya estavan creados antes de la modificación.

A continuación vamos a ver un pequeño ejemplo de como és el comportamiento.

```java
let chocolate  = {
    marca: "Chocolates LeVarez"
    precio: 2
};
```

Si mostramos la salida de este objeto a través del mètodo ```toString()``` que viene predefinido en el objeto prototype podremos observar la siguiente salida:

```java
chocolate.toString();
//Output: '[object Object]'
```
Ahora a traves de otro objeto nuevo podemos modificar el comportamiento de la funcion ```toString()```. Accederemos a el a través de la propiedad ```__proto__```:

```java
let chocolateConLeche  = {
    marca: "Chocolates LeVarez",
    precio: 3
};

chocolateConLeche.__proto__.toString = () => true;

chocolate.toString();
//Output: True
```

Ahora se ha cambiado completamente el comportamiento de la función toString en todos los objetos del programa, de tener un valor de retorno una cadena de caràcteres (String) ahora tiene un valor Boolean. Si en nuestra aplicación se canvian los estos valores podria provocar un crasheo de la apliacion.

Veamos que operaciones pueden influir en este tipo de ataques y que pasaria si un atacante pudiera modificar la propiedad property de un objeto. Según [Olivier Arteau](https://www.youtube.com/watch?v=LUsiFV3dsK8&ab_channel=NorthSec) hay algunas operaciones conflictivas que permetirian la inserción de propiedades en este objeto. La operacion ```merge(a,b)``` puede ser peligrosa.

```java
let chocolate  = {
    promocion: "50%"
    precio: 2,
};

let chocolateConMiel  = {
    precio: 4,
    extras: "miel"
};

let chocolateFusionado = merge(chocolat, chocolateConMiel);
/*Output
chocolateFusionado = {
    promocion: "50%"
    precio: 4,
    extras: "miel"
}
*/
```
Podemos observar que cuando 2 objetos comparten un mismo atributo el segundo tiene predominancia y sobrescribe el valor del primero. veamos la operacion merge:

```java
let merge = (a, b) => {
    for (let attr in b) {
        if(isObject(a[attr]) && isObject(b[attr])){
            merge(a[attr], b[attr]);
        } else {
            a[attr] = b[attr];
        }
    }
    return a;
}
```

La función recorre todos los atributos del objeto ```b```, por cada atributo comprueba si el actual es un objecto y existe en los dos, si és asi hace una llamada recursiva de la misma función, si no, substituye o añade al objeto ```a``` el atributo actual. Finalmente devuelve el objeto ```a```. Entonces si tenemos un objeto ```b``` cuyo valor és ```{"__proto__":{"contaminado":"estoy contaminado"}}```. Si éste se aplica a la función merge puede contaminar todos los objetos que ya existen y a todos los futuros objetos que se creen.

```java
let chocolate = {
    precio:  2,
}

let contaminado = {
    __proto__: {
        contaminado: "estoy contaminado"
    }
}

merge(chocolate, contaminado);

let chocolateDelVacio = {};

chocolateDelVacio.contaminado;
//Output: 'estoy contaminado'
```
Otra operación muy frecuente que puede provocar el mismo efecto és la operación ```clone(a)```. Si éste objecto se clona a través de la función ```merge({},a) también serà suceptible al prototype pollution.

```java
let clone = (a) => {
    return merge({}, a);
}

let contaminado = {
    __proto__: {
        contaminado: "estoy contaminado"
    }
}

let chocolateClonado = clone(contaminado);
chocolateClonado.contaminado;
//Output: 'estoy contaminado'
```
Otra operación seria la de la assignación por ruta o path assignment. Donde algunas librerias nos permiten assignar un valora a una propiedad de un objecto a través de una ruta tal i como se puede ver a continuación, donde el primer paràmetro és el objeto, el segundo la ruta a la propiedad i el tercero el valor:

```java
let chocolate  = {
    precio: {
        precio: 1.5,
        iva: 47
    }
};

setValue(chocolate, "precio.iva", 0);

chocolate.precio.iva
//Output: 0
```
Si los atacantes són capaces de controlar el paràmetro de ruta podrian introducir como ruta ```__proto__.contaminado``` y contaminar todos los objetos.

```java
let chocolate  = {
    precio: {
        precio: 1.5,
        iva: 47
    }
};

setValue(chocolate, "__proto__.contaminado", "estoy contaminado");

let chocolateDelVacio = {};
chocolateDelVacio.contaminado
//Output: 'estoy contaminado'
```

Por lo que nunca debemos dar al usuario control absoluto de la entrada de la ruta. Visto todo esto cuáles són los possibles ataques que se podrian realizar a través de prototype pollution?

Si pensamos, en muchos programas puede ser que haya una crida de todas propiedades de un objeto, que pasaría si una de estas propiedades fuera recursíva a si misma? Asi és! Stack Overflow!

Como puede efectuar-se esto? bien, como hemos dicho todos los objetos se crean a partir del objeto prototype, por lo que si tiene una propiedad que si contaminamos con una propiedad que solo contenga un objeto vació éste se va a contener a sí mismo recursivamente.

```java
Object.prototype.nada = {}
({}).nada.nada.nada.nada === ({}).nada;
```
Podemos observar que se puede ir llamando el objeto a si mismo sin limite. Entonces para poder añadir un objeto vacio sin que se llame recursivamente a si mismo lo que deberiamos hacer es crear un objeto con una propiedad que se llame igual que su propiedad i hacer a que apunta a "nada". De tal forma que cuando se cree un objeto este no se podria llamar recursivamente.

```java
Object.prototype.nada = {nada: ""}
({}).nada.nada === "";
```



Teniendo esto en cuenta ahora vamos a ver los AST (Abstract node tree). Los AST són la representació de árbol de la estructura sintáctica simplificada del código fuente escrito en cierto lenguaje de programación. Los AST són utilizados muy a menudo en JavaScript como plantillas de motores, typescript...

A continuación podemos ver como es el AST de nodeJS usando ```handlebears```:

![NodeJS AST](/nodejsast.png#center)

En la foto anterior podemos observar que nuestra plantilla de handlebears sigue una sèrie de passos hasta ser compilado. Tenemos nuestra plantilla que la passamos por un lexer. El lexer és una herramienta que se encarga de convertir el texto a una lista de tokens para processar posteriormente. Normalemente se usa para crear lenguajes de programación pero a veces tambien para el processamiento de texto entre otras cosas. Para mas información sobre como crear tu propia lexer pulsa [aquí](https://dev.to/areknawo/lexing-in-js-style--b5e).

Una vez tenemos los tokens el siguiente passo es el parser o analizador sintáctico. El parser és un programa que analiza una cadena de tokens según las reglas de la gramàtica del lenguaje. El parser convierte los tokens de entrada en otras estructuras (comunmente árboles). que són más útiles para el psoterior anàlisis ya que capturan la jerarquía de la entrada, estos són nombrados árboles de sintaxis abstracta.

Veamos con un ejemplo fàcil como esto funciona. Imaginemos la entrada ```12*(3+4)^2``` de una calculadora. El lexer generaria los siguientes tokens ```12, *, (, 3, +, 4, ), ^ y 2``` (análisis léxico), el parser contendrà las reglas necesarias para indicar que los simbolos ```*, (, +, ) y ^``` indican el comienzo de un nuevo token. Lo siguiente és un análisis sintáctico para comprobar que el conjunto de tokens forman una expressión vàlida y por último queda el análisis sintàctico que evalua la expressión [[ 1 ]](https://www.techopedia.com/definition/3854/parser)[[ 2 ]](https://es.wikipedia.org/wiki/Analizador_sint%C3%A1ctico). Los siguientes pasos seria la compilación del AST a lenguaje máquina i la ejecución del código.

Entonces a través de la propiedad prototype se podria insertar codigo durante las operaciones del Parser o del Compilador. En la imagen que se muestra a continuación muestra como se podria insertar parte de codigo a través del