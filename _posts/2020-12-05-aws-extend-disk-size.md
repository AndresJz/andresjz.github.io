---
title: Como crear una certifcado para acceso por ssh
layout: post
author: luisjuarez
thumbnail: ""
category: linux
summary: null
keywords: linux,ssh
permalink: "/blog/linux-ssh-certifcate"
---

Cuando trabajamos de forma remota con un servidor de aplicaciones la administración y mantenimiento del Sistema operativo suele resolverse utilizando programas de conexión remota como SSH


Si ya tenemos habilitado SSH por default el modo de autenticación se realiza por contraseña, este método no es del todo seguro ya que alguien puede tratar de adivinar la contraseña utilizando métodos como fuerza bruta.

Afortunadamente podemos utilizar certificados para mejorar esto y reducir la posibilidad de que se comprometa el acceso al servidor de nuestra aplicación.

Para crear el certificado e instalarlo existen varias formas de hacerlo, por ahora solo abordaremos una pero en futuro añadré mas opciones

Instrucciones:

1. En tu computadora (con ssh ya instalado) ejecuta 


```sh
  ssh-keygen -t rsa -b 4096 -C "test@test" -f key_test -P "test"
```

  Esto generará  el certificado para autenticación

  donde 

  * -t Es el tipo de encriptación  rsa | dsa | ecdsa 
  * -b Es el número de bits en la llave
  * -C Es el comentario que se coloca al final de la clave pública y privada
  * -f Es el nombre del archivo donde se gurdará la clave
  * -P Es la frase secreta para poder ingresar con un cerificado (Este dato es opcional pero es conveniente utilizarlo para incrementar un poco más la segridad)

    <img src="https://images.ctfassets.net/0lvk5dbamxpi/1tHf8swdhVqrjhnrNPmuhX/223abbfd94103e913a7dfc8d1b25f77f/Generating_an_SSH_key_with_ssh-keygen" class="img-fluid">

  Una vez haya concluido el proceso de generación obtendrás como resultado dos archivos, en el ejemplo anterior `key_test` `key_test.pub` (donde `key_test.pub `es la clave pública)
  
  Más opciones de configuración las puedes visualizar en este enlace: https://www.ssh.com/ssh/keygen/

  *Nota*  Guarda el FingerPrint generado para futuras validaciones 
  

2. Si quieres validar que la clave privada es valida tu puedes correr el siguiente comando, el cual te pedira la frase secreta para verificar que el certifcado no fue alterado.

```sh
ssh-keygen -y -f key_test 
```
  
3. El último paso consiste en copiar la clave pública a nuestro certicado  

  La forma más sencilla de lograr el copiado es utilizando un comando ya existente para el copiado automático, sin embargo puedes hacerlo también de formal manual aunque deberías de hacerlo con precaución
  
  Manual
  
  * Obtener el contenido del archivo publico,  esto se puede realizar corriendo`cat key_test.pub`
  * Una vez se tenga el contenido puedes agregar el contenido al siguiente archivo `$HOME/.ssh/authorized_keys` (donde home es la ríz del usuario)
  * Finalmente valida que puedas ejecutar el inicio de sesión utilizando el certificado previamente creado.

  Automática (Recomendado)
  * No hay mucho que decir en este caso, solo verifca que el path del arhivo a subir es correcto, y finalmente corre el comando

```sh
ssh-copy-id -i key_test.pub <user>@<remote>
```

  En cualquiera de los casos una vez configurado nuestro certificado en el folder de `authorized_keys` del servidor remoto, podemos acceder desde nuestro ambiente local utilizando el certificado generado.

```sh
ssh -i key_test <user>@<remote>
```


4. Finalmente como recomendación, no olvides rotar las credenciales de forma regular para mantener seguro el sistema y evitar que el certificado sea utilizado por alguién ajeno.





  
  
  





