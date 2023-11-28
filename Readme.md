## AJAX: Asynchronous JavaScript y XML

Antes de comenzar con lo pedido vamos a ver si todo se resuelve correctamente para esto ejecutamos `bundle install` y posteriormente `rails server` y verificamos en la pagina web local.

![](https://github.com/Kinartb/Ajax/blob/main/imagenes/1.png)

Vemos que aparece un error que indica que aun no se ha creado las mgiraciones, una vez realizada la migracion con `rails db:migrate RAILS_ENV=development`, posteriormente habra otro error que diga que no se encuentra la tabla `moviegoes` y luego otro error de indección donde se encuentra un `end` de más. Se solucionan los errores y se escribe `rails server` y veremos los resultados en la pagina web local.

![](https://github.com/Kinartb/Ajax/blob/main/imagenes/2.png)

Una vez solucionadas nuestros problemas podemos ir a realizar la actividad.


### Parte 1

El paso 1 necesita que identifiquemos o creemos una nueva acción de controlador que será la encargada de gestionar la petición. Usaremos la acción ya existente `MoviesController#show`, por lo que no necesitaremos definir una nueva ruta. Esta decisión de diseño es justificable, dado que la versión AJAX de la acción realiza la misma función que la versión original, es decir, la acción REST de mostrar (`show`). 

Modifica la acción `show` de forma que, si está respondiendo a una petición AJAX, procesará la sencilla vista parcial el código siguiente en lugar de la vista completa.

Este codigose implementa en `app/views/movies/show.html.erb`

```erb
 <p> <%= movie.description %> </p>
 <%= link_to 'Edit Movie', edit_movie_path(movie), :class => 'btn btn-primary' %>
 <%= link_to 'Close', '', :id => 'closeLink', :class => 'btn btn-secondary' %>
```

¿Cómo sabe la acción de controlador si `show` fue llamada desde código JavaScript o mediante una petición HTTP normal iniciada por el usuario? Utiliza el código siguiente  para mostrar la acción del controlador que renderizará la vista parcial. 

Este codigo es que se va a implementar en `movies_controller.rb`

```ruby
class MoviesController < ApplicationController
  def show
    id = params[:id] # retrieve movie ID from URI route
    @movie = Movie.find(id) # look up movie by unique ID
    render(:partial => 'movie', :object => @movie) if request.xhr?
    # will render app/views/movies/show.<extension> by default
  end
end
```
A continaucion ejecutamos  `rails server` y vemos que todo nos sale normal pero al momento de dar clic en  `Edit movie` vemos que nos sale un error

Este error indica que el controlador MoviesController no tiene una plantilla (template) asociada para el formato de solicitud text/html.

### Parte 2

 ¿Cómo debería construir y lanzar la petición XHR el código JavaScript? Queremos que la ventana flotante aparezca cuando pinchamos en el enlace que tiene el nombre de la película.

Explica el siguiente código

```javascript
var MoviePopup = {
  setup: function() {
    // add hidden 'div' to end of page to display popup:
    let popupDiv = $('<div id="movieInfo"></div>');
    popupDiv.hide().appendTo($('body'));
    $(document).on('click', '#movies a', MoviePopup.getMovieInfo);
  }
  ,getMovieInfo: function() {
    $.ajax({type: 'GET',
            url: $(this).attr('href'),
            timeout: 5000,
            success: MoviePopup.showMovieInfo,
            error: function(xhrObj, textStatus, exception) { alert('Error!'); }
            // 'success' and 'error' functions will be passed 3 args
           });
    return(false);
  }
  ,showMovieInfo: function(data, requestStatus, xhrObject) {
    // center a floater 1/2 as wide and 1/4 as tall as screen
    let oneFourth = Math.ceil($(window).width() / 4);
    $('#movieInfo').
      css({'left': oneFourth,  'width': 2*oneFourth, 'top': 250}).
      html(data).
      show();
    // make the Close link in the hidden element work
    $('#closeLink').click(MoviePopup.hideMovieInfo);
    return(false);  // prevent default link action
  }
  ,hideMovieInfo: function() {
    $('#movieInfo').hide();
    return(false);
  }
};
$(MoviePopup.setup);
```
*RPTA*:  lo que realiza el siguiente código es que proporciona una forma de mostrar información de películas en una ventana cuando el usuario hace clic en enlaces con el ID movies. Utiliza AJAX para obtener datos de la película y luego muestra estos datos en una ventana emergente.

Ocurren algunos trucos interesantes de CSS en el código anterior Puesto que el objetivo es que la ventana emergente `flote`, podemos utilizar CSS para especificar la posición como absolute añadiendo el siguiente código en `app/assets/stylesheets/application.css` :

```css
#movieInfo {
  padding: 2ex;
  position: absolute;
  border: 2px double grey;
  background: wheat;
}
```

¿Cuáles son tus resultados?

*RPTA*: Por el momento al ejecutar `rails server` teniendo cuenta el error anterior y editar la vista.

![](https://github.com/Kinartb/Ajax/blob/main/imagenes/4.png)

![](https://github.com/Kinartb/Ajax/blob/main/imagenes/5.png)

![](https://github.com/Kinartb/Ajax/blob/main/imagenes/6.png)

![](https://github.com/Kinartb/Ajax/blob/main/imagenes/7.png)

### Parte 3

Conviene mencionar una advertencia a considerar cuando se usa JavaScript para crear nuevos elementos dinámicamente en tiempo de ejecución, aunque no surgió en este ejemplo en concreto. Sabemos que `$(.myClass).on(click,func)` registra `func` como el manejador de eventos de clic para todos los elementos actuales que coincidan con la clase CSS myClass. Pero si se utiliza JavaScript para crear nuevos elementos que coincidan con `myClass` después de la carga inicial de la página y de la llamada inicial a `on`, dichos elementos no tendrán el manejador asociado, ya que `on` sólo puede asociar manejadores a elementos existentes. 

¿Cuál es solución que brinda jQuery  a este problema? 

*RPTA*: jQuery proporciona el método on con la delegación de eventos. Para el caso de `MoviePopup`. Se puede reescribir parte del código como sigue:

```javascript
$(document).on('click', '#movies a', MoviePopup.getMovieInfo);
```

Aquí, *$(document)* es el contenedor existente, *'click'* es el tipo de evento, *#movies* a es el selector dinámico que apunta a los enlaces dentro de *#movies*, y *MoviePopup.getMovieInfo* es el manejador de eventos.
