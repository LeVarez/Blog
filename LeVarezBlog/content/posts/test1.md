+++
draft = false
date = 2020-10-08T12:37:40+01:00
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

Teniendo esto en cuenta ahora vamos a ver los AST (Abstract node tree). Los AST són la representació de árbol de la estructura sintáctica simplificada del código fuente escrito en cierto lenguaje de programación. Los AST són utilizados muy a menudo en JavaScript como plantillas de motores, typescript...

A continuación podemos ver como es el AST de nodeJS:

![Example image](/test.png)