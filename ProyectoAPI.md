# Realización de API

Hecho por: Raúl Castro García

## Creación del proyecto

Lo primero que tendremos que hacer es crear el proyecto de laravel donde vamos a tener nuestra API.
Para ello, escribimos en una terminal lo siguiente:

```bash
  laravel new nombre_Proyecto 
``` 
Las opciones a elegir son las siguientes:
1. No Starter Kit 
2. PHPUnit
3. Yes (Si queremos añadir repositorio a GIT)
4. MySQL (Para la base de datos)

Una vez creado, accedemos al nuevo proyecto desde PHPStorm. 

Queremos crear un modelo para la utilización de la API, así que escribimos lo siguiente en la terminal:

```bash
  php artisan make:model Nombre_Singular --api -fm  
```     
- --api: Esta opción especifica que el modelo generado debería estar centrado en API. Esto significa que Laravel generará código base adecuado para construir una API RESTFUL,
- -fm: Para crear el factory y el migrate

Si todo ha ido bien, nos creará los siguientes archivos:

![img.png](../img.png) 

## Creación de .env y docker-compose.yaml

En este proyecto, necesitaremos estos dos archivos para montar la base de datos con sus permisos y configuraciones, ya que trabajamos con Docker.

Tendremos los archivos en nuestro proyecto de laravel.

### .env

```php
    APP_NAME=Laravel
	APP_ENV=local
	APP_KEY=base64:2LsZA1WpEGaIoFovyisJ8NXwi+oFyqCmn9nRmLk/Sxg=
	APP_DEBUG=true
	APP_URL=http://localhost

	LOG_CHANNEL=stack
	LOG_DEPRECATIONS_CHANNEL=null
	LOG_LEVEL=debug

	DB_CONNECTION=mysql
	DB_HOST=127.0.0.1
	DB_PORT=23306
	DB_DATABASE=instituto
	DB_USERNAME=alumno
	DB_PASSWORD=alumno
	DB_PASSWORD_ROOT=root
	DB_PORT_PHPMYADMIN=8080

	BROADCAST_DRIVER=log
	CACHE_DRIVER=file
	FILESYSTEM_DISK=local
	QUEUE_CONNECTION=sync
	SESSION_DRIVER=file
	SESSION_LIFETIME=120

	MEMCACHED_HOST=127.0.0.1

	REDIS_HOST=127.0.0.1
	REDIS_PASSWORD=null
	REDIS_PORT=6379

	MAIL_MAILER=smtp
	MAIL_HOST=mailpit
	MAIL_PORT=1025
	MAIL_USERNAME=null
	MAIL_PASSWORD=null
	MAIL_ENCRYPTION=null
	MAIL_FROM_ADDRESS="hello@example.com"
	MAIL_FROM_NAME="${APP_NAME}"

	AWS_ACCESS_KEY_ID=
	AWS_SECRET_ACCESS_KEY=
	AWS_DEFAULT_REGION=us-east-1
	AWS_BUCKET=
	AWS_USE_PATH_STYLE_ENDPOINT=false

	PUSHER_APP_ID=
	PUSHER_APP_KEY=
	PUSHER_APP_SECRET=
	PUSHER_HOST=
	PUSHER_PORT=443
	PUSHER_SCHEME=https
	PUSHER_APP_CLUSTER=mt1

	VITE_APP_NAME="${APP_NAME}"
	VITE_PUSHER_APP_KEY="${PUSHER_APP_KEY}"
	VITE_PUSHER_HOST="${PUSHER_HOST}"
	VITE_PUSHER_PORT="${PUSHER_PORT}"
	VITE_PUSHER_SCHEME="${PUSHER_SCHEME}"
	VITE_PUSHER_APP_CLUSTER="${PUSHER_APP_CLUSTER}"

	LANG_FAKE ="es_ES"
```

### docker-compose.yaml

```php
#Nombre de la version
version: "3.8"
services:
  mysql:
    # image: mysql <- Esta es otra opcion si no hacemos el build
    image: mysql

    # Para no perder los datos cuando destryamos el contenedor, se guardara en ese derectorio
    volumes:
      - ./datos:/var/lib/mysql
    ports:
      - ${DB_PORT}:3306
    environment:
      - MYSQL_USER=${DB_USERNAME}
      - MYSQL_PASSWORD=${DB_PASSWORD}
      - MYSQL_DATABASE=${DB_DATABASE}
      - MYSQL_ROOT_PASSWORD=${DB_PASSWORD_ROOT}

  phpmyadmin:
    image: phpmyadmin
    container_name: phpmyadmin  #Si no te pone por defecto el nombre_directorio-nombre_servicio
    ports:
      - ${DB_PORT_PHPMYADMIN}:80
    depends_on:
      - mysql
    environment:
      PMA_ARBITRARY: 1 #Para permitir acceder a phpmyadmin desde otra maquina
      PMA_HOST: mysql
```

Ahora vamos a arrancar el servicio establecido por docker. Primero tenemos que parar todos los servicios que estén en marcha.

```bash
    docker stop $(docker ps -a -q)  
```
Si queremos evitar conflictos con un contenedor "phpmyadmin" que ya esté creado, deberemos borrarlo con el siguiente comando:

```bash
    docker rm $(docker ps -a -q)
```
Si tenemos todo hecho, vamos a levantar el contenedor de la siguiente forma:

```bash
    docker compose up -d  
```
- Con -d nos permite seguir escribiendo en la terminal

Cuando tengamos el contenedor levantado, vamos a utilizar artisan para levantar el back y que el proyecto esté operativo

```bash
    php artisan serve
```

## Población de la base de datos

Tenemos la base de datos creada, pero no tenemos nada de contenido dentro de ella, así que vamos a poblarla de la siguiente forma:

### Migrations
Es el lugar donde se da la estructura de la tabla que queremos crear.

Accedemos a database/migrations/2024_02_27_093729_create_alumnos_table.php y lo modificamos:

```php
    Schema::create('nombre_plural', function (Blueprint $table) {
            $table->id();
            $table->string("nombre");
            $table->string("direccion");
            $table->string("email");
            $table->timestamps();
    });
```

### Factory
En este espacio, estableceremos los datos que va contener la tabla estructurada anteriormente.

Accedemos a database/factories/AlumnoFactory.php y lo modificamos:

```php
        public function definition(): array
        {
                return [
                "nombre" =>fake()->name(),
                "direccion" =>fake()->address(),
                "email" =>fake()->email()
                //
                ];
        }
```

* fake() nos sirve para añadir datos aleatorios

### Seeders
En este espacio recogeremos los datos de Factory para crear semillas de ello (por ejemplo podremos crear 20 filas de datos de Alumno)

Accedemos a database/seeders/DatabaseSeeder.php y ponemos lo siguiente:

```php
    Nombre::factory(20)->create();
```

Vamos a *config/app.php* para cambiar el idioma a la hora de crear los nombres (En este caso)

```php
    'faker_locale' => 'es_ES',  
```

Por último, tenemos que efectuar los cambios en nuestra base de datos, así que escribiremos el siguiente comando en la terminal:

```bash
    php artisan migrate --seed 
```     

Si todo ha ido bien, podemos observar que se ha poblado y creado la tabla alumnos:

![img_1.png](../img_1.png)


## Representación de la API

Si queremos que la representación de cada recurso siga la especificación vista,
vamos a crear un resource para adaptarlo. Con estas clases vamos a poder adaptar lo que enviamos al cliente:

```bash 
    php artisan make:request AlumnoFormRequest
    php artisan make:resource AlumnoResource
    php artisan make:resource AlumnoCollection
```

Podemos ver que se han creado en sus respectivos directorios

![img_2.png](../img_2.png)

- AlumnoFormRequest: Crea una clase de solicitud que se utiliza para validar y procesar datos de entrada en una solicitud HTTP.

- AlumnoResource: Genera una clase de recurso para personalizar la presentación de un modelo en respuestas de API, permitiendo controlar qué campos se incluyen y aplicar transformaciones a los datos.

- AlumnoCollection: Crea una clase de colección de recursos para personalizar la presentación de una colección de modelos en respuestas de API, permitiendo controlar el formato de la colección y de los elementos.

Dentro de AlumnoCollection, modificamos la función with para agregar datos adicionales al resultado de una respuesta JSON.

```php
public function with(Request $request)
    {
    return[
        "jsonapi" => [
            "version"=>"1.0"
        ]
    ];
}
```

Ahora en AlumnoResource, crearemos la estructura de los datos de la API. La estructura es la siguiente:

```php
public function toArray(Request $request): array{
    return[
        "id"=>$this->id,
        "type" => "Nombre",
        "attributes" => [
            "nombre"=>$this->nombre,
            "direccion"=>$this->direccion,
            "email"=>$this->email,
        ],
        "link"=>url('api/nombre_plural'.$this->id)
    ];
}
```

### Rutas

Tenemos que establecer la ruta de la API creada para que se muestre, y hay que hacerlo de diferente forma que con blade ya que no se está utilizando un servidor de front.

Accedemos a routes/api.php y escribimos lo siguiente:

![img_3.png](../img_3.png)

Ahora podremos comprobar las rutas que hemos creado con el siguiente comando:

```bash 
    php artisan route:list --path='api/alumnos'
```

![img_4.png](../img_4.png)

Podemos comprobar que las restricciones RESTFUL se han aplicado correctamente ya que se utilizan PUT, PATCH...

## Adaptando el JSON al formato JSON:API spec (Parte 1)

Ahora el index, llamará a Collection que retornará la colección (collection), pero de
forma implícita cada elemento de la colección será adaptado a un recurso

Vamos a AlumnoController y modificamos el index de la siguiente forma:

```php
public function index()
{
    $nombre_plural = Nombre::all();
    return response()->json($nombre_plural);
}
```

La función index se utiliza para obtener y devolver una lista de recursos (en este caso, alumnos) en el contexto de una API. 

Ahora modificaremos el show() de la siguiente forma:

```php
    public function show(int $id)
    {
        //
        // return ($alumno);
        $alumno = Alumno::find($id);
        if (!$alumno){
            return response()->json([
                "errors"=>[
                    "status"=>404,
                    "title"=>"Resource not found",
                    "details"=>"$id Alumno not found"
                ]
            ],404);
        }
        return new AlumnoResource($alumno);
    }
```

La función show la utilizaremos para realizar peticiones GET, de manera que obtendremos los datos del alumno buscado o de todos en general.

El resultado es un ALumnoResource con el formato establecido anteriormente.

Si no se encuentra el alumno, saltará una excepción con códgio 404, que significa "Not Found".

## Manejo de errores

### Error de base de datos

Declaramos el método render de la clase Exceptions/Handler.php para que nos muestre un error de base de datos si no ha sido capaz de conectarse.

```php
public function render($request, Throwable $exception)
    {
        if ($exception instanceof ValidationException)
            return $this->invalidJson($request, $exception);
        // Errores de base de datos)
        if ($exception instanceof QueryException) {
            return response()->json([
                'errors' => [
                    [
                        'status' => '500',
                        'title' => 'Database Error',
                        'detail' => 'Error procesando la respuesta. Inténtelo más tarde.'
                    ]
                ]
            ], 500);
        }
        // Delegar a la implementación predeterminada para otras excepciones no manejadas
        return parent::render($request, $exception);
    }
```

Si paramos el contenedor de Docker con la base de datos, podremos ver que nos salta el error establecido:

![img_5.png](../img_5.png)

### Middleware

Los middlewares son capas intermedias que pueden realizar acciones antes o después de que una solicitud llegue a su destino final, como un controlador. 

En este caso específico, el middleware está destinado a verificar y manejar la cabecera 'accept' en la solicitud para garantizar que se espera y acepta un tipo de contenido específico (application/vnd.api+json), que es comúnmente utilizado en API JSON:API.

Creamos el Middleware deseado de la siguiente forma:

```bash 
php artisan make:middleware HandleMiddleware
```

Ahora lo encontramos dentro de app/http/middleware y lo modificamos:

```php  
public function handle(Request $request, Closure $next): Response
{
    if ($request->header('accept') != 'application/vnd.api+json') {
        return response()->json([
            "errors"=>[
                "status"=>406,
                "title"=>"Not Accetable",
                "deatails"=>"Content File not specifed"
            ]
        ],406);
    }
    return $next($request);
}
```
Por último, tenemos que asociarlo al kernel.php para que se active:

app/http/kernel.php

![img_6.png](../img_6.png)

Ahora ya no podemos ver el resultado de la API como antes desde el navegador ya que no tenemos activado el nuevo Accept.

### Validación de formularios

Si hay un error en la validación del formulario se va a gestionar una excepción de de validación
ValidationException
Para gestionarla, en la clase Handler reescribimos el método invalidJson (debe devolver un
JsonResponse)

```php
   protected function invalidJson($request, ValidationException $exception):JsonResponse
    {
        return response()->json([
            'errors' => collect($exception->errors())->map(function ($message, $field) use
            ($exception) {
                return [
                    'status' => '422',
                    'title'  => 'Validation Error',
                    'details' => $message[0],
                    'source' => [
                        'pointer' => '/data/attributes/' . $field
                    ]
                ];
            })->values()
        ], $exception->status);
    }
```

## Adaptando el JSON al formato JSON:API spec (Parte 2)

Vamos a establecer el método store para realizar las peticiones POST y poder añadir nuevo contenido a nuestra API

La estructura es la siguiente:

Primero vamos a crear el fillable de Alumno desde el mismo modelo:

```php
    protected $fillable = ["nombre", "direccion", "email"];
```

Después, establecemos las reglas de validación para cada attributo a introducir, entonces usaremos AlumnoFormRequest:

```php
public function authorize(): bool
{
    return true;
}

public function rules(): array
{
    return [
            "data.attributes.nombre"=>"required|min:3",
            "data.attributes.direccion"=>"required",
            "data.attributes.email"=>"required|email|unique:alumnos,email"
            //
        ];
}
```

La función authorize permite que se puedan utilizar las reglas establecidas.

data.attributes coge los valores del JSON que han sido enviados.

Vemos que nuestro valor único es el email, es decir, que no se puede repetir, y posteriormente lo usaremos para manejar las excepciones

Por último, modificamos la función Store de AlumnoController con lo establecido para que nos cree una nueva instancia de AlumnoResource y su respectivo esqueleto.

```php
    public function store(AlumnoFormRequest $request)
    {
        $datos = $request->input("data.attributes");
        $alumno = new Alumno($datos);
        $alumno->save();
        return new AlumnoResource($alumno);
    }
```

Ahora podemos comprobar que se pueden hacer peticiones POST, por lo que vamos a POSTMAN, y lo primero que vamos a hacer es crear un nuevo header para manejar el middleware establecido anteriormente

![img_7.png](../img_7.png)

Una vez hecho, probamos las dos peticiones que tenemos establecidas.

Introducimos la url - http://localhost:8000/api/alumnos

GET:

![img_8.png](../img_8.png)

POST:

Introducimos un nuevo contenido, y si al mandar los datos no sale ningún error, es que lo hemos hecho bien.

![img_9.png](../img_9.png)

Si volvemos a introducir un mismo email, nos dará un error 422 de error de validación y no nos dejará crear uno nuevo puesto que ya existe ese email.

![img_10.png](../img_10.png)

## Adaptando el JSON al formato JSON:API spec (Parte 3)

Nos quedan los métodos de DELETE y PUT/PATCH, así que vamos a establecerlos.

### DELETE

Modificamos la función destroy de la siguiente forma:

```php
public function destroy(int $id)
    {
        $alumno = Alumno::find($id);
        if (!$alumno){
            return response()->json([
                "errors"=>[
                    "status"=>404,
                    "title"=>"Resource not found",
                    "details"=>"$id Alumno not found"
                ]
            ],404);
        }

        $alumno->delete();
        return response()->json(null, 204);
        //
    }
```
Lo único que hacemos aquí es buscar la id del alumno introducido. Si no lo encuentra nos saltará la excepción correspondiente, y si lo encuentra nos eliminará el alumno.

![img_11.png](../img_11.png)

Ahora con GET comprobamos que ya no está:

![img_12.png](../img_12.png)

### UPDATE

Habrá dos formas de realizar un UPDATE: 

-  PUT: Es obligatorio poner todos los datos aunque no se quieran modificar. Es el equivalente a reemplazar lo que había antes.
-  PATCH: Solo hay que poner lo que se quiera modificar. Es el equivalente a hacer un parche.

Una vez entendido esto, vamos a modificar la función update:

```php
public function update(Request $request, int $id)
    {
        //

        $alumno = Alumno::find($id);
        if (!$alumno) {
            return response()->json([
                "errors" => [
                    "status" => 404,
                    "title" => "Resource not found",
                    "details" => "$id Alumno not found"
                ]
            ], 404);
        }
        $verbo = $request->method();
        //En función del verbo creo unas reglas de
        // validación u otras
        if ($verbo == "PUT") { //Valido por put
            $rules = [
                "data.attributes.nombre" => ["required", "min:5"],
                "data.attributes.direccion" => "required",
                "data.attributes.email" => ["required", "email",
                    Rule::unique("alumnos", "email")
                        ->ignore($alumno)]
            ];

        } else { //Valido por PATCH
            if ($request->has("data.attributes.nombre"))
                $rules["data.attributes.nombre"]= ["required", "min:5"];
            if ($request->has("data.attributes.direccion"))
                $rules["data.attributes.direccion"]= ["required"];
            if ($request->has("data.attributes.email"))
                $rules["data.attributes.email"]= ["required", "email",
                    Rule::unique("alumnos", "email")
                        ->ignore($alumno)];
        }

        $datos_validados = $request->validate($rules);
        foreach ($datos_validados['data']['attributes'] as $campo =>$valor)
            $datos[$campo] = $valor;

        $alumno->update($datos);
        return new AlumnoResource($alumno);
    }
```

Puntos a destacar:
- Declaramos una variable $verbo para recoger si se está haciendo la petición por PUT o por PATCH
- Si es por PUT, tenemos que verificar todo el contenido de golpe, y si es por PATCH, realizaremos condiciones para que el sistema compruebe lo que se está queriendo modificar
- Rule::unique es una clase proporcionada por Laravel que se utiliza para definir reglas de validación de unicidad en la base de datos. En el contexto de este código, se está utilizando para validar que el valor de un campo (email) sea único en la tabla de alumnos, excepto para el alumno actual que se está actualizando.
- Se validan los datos de la solicitud con las reglas aplicadas anteriormente
- Se hace un foreach para actualizar los datos del alumno correspondiente
- Por último se devuelve el ALumnoResource con las modificaciones hechas.

Comprobamos el funcionamiento en POSTMAN:

### PUT

Si modificamos todo el contenido nos muestra el resultado y sin errores

![img_13.png](../img_13.png)

Intentando modificar solo un contenido nos saltan los errores:

![img_14.png](../img_14.png)

Por último, si quiero meter otro correo que ya está metido, nos da error:

![img_15.png](../img_15.png)

### PATCH

Modificamos solo un valor:

![img_16.png](../img_16.png)

Si intentamos modificar un correo que ya existe, no nos deja:

![img_17.png](../img_17.png)







