---
title: Incrementar el tamaño de disco en AWS EC2
layout: post
author: luisjuarez
thumbnail: "/assets/img/posts/hello.jpg"
category: aws
summary: null
keywords: aws
permalink: "/blog/aws-extend-disk-size"
---

1. El primer punto consiste en Ir a la sección de aws de EC2 y seleccionar la instancia a la cual queremos aumentar el tamaño, asegurate de que este en estado detenida antes de continuar

<img src="/assets/img/posts/aws-size-29-11/1.png" class="img-fluid">

Como puedes observar  en la parte inferior se muestran alguna pestañas, deberás elegir la de almacenamiento y posteriormente dar clic en el volumen correspondiente.

2. Selecciona el volumen y posteriormente da clic en modificar volumen, tal como se muestra a continuación

<img src="/assets/img/posts/aws-size-29-11/2.png" class="img-fluid">

3.  Se mostrará un modal con un formulario para hacer el ajuste de tipo de volumen y tamaño, puedes cambiar el tipo solo considera que hay un costo de por medio por lo que por ahora solo modificaremos el tamaño de 30 GB a 50 GB


<img src="/assets/img/posts/aws-size-29-11/3.png" class="img-fluid">

4. Una vez realizada la modificación verás que el estado cambia a "in-use-optimizing" lo que significa que  la configuiración se esta aplicando.

<img src="/assets/img/posts/aws-size-29-11/4.png" class="img-fluid">

5. Finalmente, una vez que el estatus vuelva a activo podrás iniciar la instancia y validar que el tamaño cambio

```python
// detalles de disco
df -h

```
