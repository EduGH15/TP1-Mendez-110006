<div align="right">
<img width="32px" src="img/algo2.svg">
</div>

# TP1

## Repositorio de (Eduardo González) - (110006) - (eegonzalez@fi.uba.ar)

- Para compilar:

```bash
make pruebas_chanutron
```

- Para ejecutar:

```bash
./pruebas_chanutron
```

- Para ejecutar con valgrind:
```bash
make valgrind-chanutron
```
---
##  Funciónamiento de: informacion_pokemon_t *pokemon_cargar_archivo(const char *path)

El programa funciona abriendo el archivo pasado como parámetro y leyendolo línea por línea. Si la línea es válida, carga los datos en el struct. Dependiendo de en qué linea se encuentre el error se mantendrá o se restará la cantidad de pokemones válidos presentes en el struct de informacion_pokemon: en caso de que el error se encuentre en la linea de pokemon, no se modificará la cantidad de pokemones. Si el error se encuentra en las lineas correspondientes al ataque, se restará la cantidad de pokemones presentes en el struct.
Ejemplo: 

Error en la línea de ataque:

Pikachu;E -> Esta linea es válida. Se agrega al struct y aumenta la cantidad de pokemones.
Rayo;E;5 -> Esta linea también es válida. Se agrega al struct y aumenta la cantidad de ataques.
Latigo;N;1 -> Esta linea también es válida. Se agrega al struct y aumenta la cantidad de ataques.
Chispa;E:1 -> Esta linea no es válida, por lo tanto el pokemon que agregué no es válido asi que se resta la cantidad de pokemones y dejo de leer el archivo.

Error en la linea de pokemon:

Pikachu:E -> Esta linea no es valida, asi que nunca aumento la cantidad de pokemones y dejo de leer el archivo. 
Rayo;E;5
Latigo;N;1
Chispa;E;1

##  Flujo de la función:

<div align="center">
<img width="70%" src="img/Diagrama de flujo.png">
</div>

##  Diagramas de memoria:

En el archivo `pokemon.c` la función `pokemon_cargar_archivo` utiliza `malloc` para reservar memoria. Primero se reserva memoria para el struct de información, en caso de que no se pueda, se corta el programa. Luego se reserva memoria para el struct de pokemon, en caso de que no se pueda, se libera la memoria de informacion_pokemon y se corta el programa.

```c
struct info_pokemon *info = malloc(sizeof(struct info_pokemon));
	if (info == NULL) {
		fclose(archivo);
		return NULL;
	}
	info->cantidad = SIN_POKEMON;

	info->pokemones = malloc(sizeof(struct pokemon));
	if (info->pokemones == NULL) {
		free(info);
		fclose(archivo);
		return NULL;
	}
```

El diagrama sería el siguiente:

<div align="center">
<img width="70%" src="img/Memoria 1.png">
</div>

En el archivo `pokemon.c` la función `asignar_pokemon` utiliza `realloc` para aumentar el espacio de memoria. El resultado de realloc lo guardo en una variable auxiliar para verificar si se pudo o no redimensionar el espacio. En caso de que se pueda entonces el vector original será igual al auxiliar. 

```c
pokemon_t *aux =
		realloc(info->pokemones,
			sizeof(struct pokemon) *
				(long unsigned int)(info->cantidad + 1));
	if (aux != NULL) {
		info->pokemones = aux;
	}
```

El diagrama sería el siguiente: 

<div align="center">
<img width="70%" src="img/Memoria 2.png">
</div>


## Complejidad:

Con respecto a las funciones `inicializar_ataque`, `inicializar_pokemon` y `quitar_salto_linea` tienen complejidad O(1) ya que las primeras dos funciones son de tiempo de ejecución constante y `quitar_salto_linea` depende de la cantidad de caracteres presentes en la linea por lo que seria O(n).

Las funciones `es_tipo_valido` y `asignar_tipo` son de tiempo constante asi que su notación será O(1).

La complejidad de `ordenar_pokemones` es de O(n^2) ya que se usa el algoritmo de ordenamiento de Bubble sort.

```c
void ordenar_pokemones(pokemon_t *pokemones, int cantidad_pokemones)
{
	if (pokemones == NULL) {
		return;
	}

	bool ordenado = false;
	int i = 0;
	pokemon_t aux;
	while (i < cantidad_pokemones - 1 && !ordenado) {
		ordenado = true;
		for (int j = 0; j < cantidad_pokemones - i - 1; j++) {
			if (strcmp(pokemones[j].nombre,
				   pokemones[j + 1].nombre) > 0) {
				aux = pokemones[j];
				pokemones[j] = pokemones[j + 1];
				pokemones[j + 1] = aux;
				ordenado = false;
			}
		}
		i++;
	}
	return;
}
```

En este caso la mayor parte de las operaciones es de tiempo constante a excepcion de strcpy que tiene una complejidad de O(n) ya que depende de la longitud del nombre. En definitiva, la complejidad de la función `asignar_pokemon` es de O(1) ya que las operaciones que nos interesan con constantes con respecto a la cantidad de pokemones.

```c
void asignar_pokemon(informacion_pokemon_t *info, char nombre_pokemon[20],
		     char tipo_pokemon)
{
	if (info == NULL) {
		return;
	}
	inicializar_pokemon(info->pokemones, info->cantidad);
	(info->cantidad)++;
	strcpy(info->pokemones[info->cantidad - 1].nombre, nombre_pokemon);
	asignar_tipo(&(info->pokemones[info->cantidad - 1]), tipo_pokemon);
	pokemon_t *aux =
		realloc(info->pokemones,
			sizeof(struct pokemon) *
				(long unsigned int)(info->cantidad + 1));
	if (aux != NULL) {
		info->pokemones = aux;
	}
}
```

En este caso la mayor parte de las operaciones es de tiempo constante a excepcion de strcpy que tiene una complejidad de O(n) ya que depende de la longitud del nombre. En definitiva la complejidad de la función `asignar_ataque` es de O(1) ya que las operaciones que nos interesan son constantes con respecto a la cantidad de ataques.

```c
void asignar_ataque(informacion_pokemon_t *info, char nombre_ataque[20],
		    char tipo_movimiento, unsigned int poder)
{
	if (info == NULL) {
		return;
	}
	(info->pokemones[info->cantidad - 1].cantidad_ataques)++;
	strcpy(info->pokemones[info->cantidad - 1]
		       .ataques[info->pokemones[info->cantidad - 1]
					.cantidad_ataques -
				1]
		       .nombre,
	       nombre_ataque);
	asignar_tipo(&(info->pokemones[info->cantidad - 1]
			       .ataques[info->pokemones[info->cantidad - 1]
						.cantidad_ataques -
					1]),
		     tipo_movimiento);
	info->pokemones[info->cantidad - 1]
		.ataques[info->pokemones[info->cantidad - 1].cantidad_ataques -
			 1]
		.poder = poder;
}
```

La complejidad de la función `pokemon_buscar` es de O(n) ya que depende del tamaño de los datos, en este caso el tamaño de los datos viene dado por la cantidad de lineas que se encuentren presentes al leer el archivo. 
```c
informacion_pokemon_t *pokemon_cargar_archivo(const char *path)
{
	if (path == NULL) {
		return NULL;
	}

	FILE *archivo = fopen(path, "r");
	if (!archivo) {
		return NULL;
	}

	struct info_pokemon *info = malloc(sizeof(struct info_pokemon));
	if (info == NULL) {
		fclose(archivo);
		return NULL;
	}
	info->cantidad = SIN_POKEMON;

	info->pokemones = malloc(sizeof(struct pokemon));
	if (info->pokemones == NULL) {
		free(info);
		fclose(archivo);
		return NULL;
	}

	info->pokemones[info->cantidad].cantidad_ataques = SIN_ATAQUES;

	char linea[MAX_CARACTERES];
	bool seguir_leyendo = true;
	char nombre_pokemon[MAX_NOMBRE];
	char tipo_pokemon;
	char nombre_ataque[MAX_NOMBRE];
	char tipo_movimiento;
	unsigned int poder;
	int cantidad_pokemones_validos = SIN_POKEMON;
	int linea_datos = 0;

	while (fgets(linea, sizeof(linea), archivo) != NULL && seguir_leyendo) {
		int cantidad_campos_invalidos = SIN_CAMPOS_INVALIDOS;
		int cantidad_delimitadores = SIN_DELIMITADORES;
		quitar_salto_linea(linea);

		for (size_t i = 0; linea[i] != '\0'; i++) {
			if (linea[i] == ':') {
				cantidad_campos_invalidos++;
			} else if (linea[i] == ';') {
				cantidad_delimitadores++;
			}
		}

		if (cantidad_delimitadores == 1 &&
		    cantidad_campos_invalidos == 0) {
			sscanf(linea, "%[^;];%c", nombre_pokemon,
			       &tipo_pokemon);
			if (es_tipo_valido(tipo_pokemon)) {
				asignar_pokemon(info, nombre_pokemon,
						tipo_pokemon);
				cantidad_pokemones_validos++;
			} else {
				cantidad_campos_invalidos++;
			}
		} else if (cantidad_delimitadores == 2) {
			sscanf(linea, "%[^;];%c;%u", nombre_ataque,
			       &tipo_movimiento, &poder);
			if (es_tipo_valido(tipo_movimiento)) {
				asignar_ataque(info, nombre_ataque,
					       tipo_movimiento, poder);
			} else {
				cantidad_campos_invalidos++;
			}
		}

		if (cantidad_campos_invalidos != SIN_CAMPOS_INVALIDOS) {
			if (linea_datos != 0) {
				cantidad_pokemones_validos -= 1;
			}
			info->cantidad = cantidad_pokemones_validos;
			seguir_leyendo = false;
		}

		if (linea_datos == 3) {
			linea_datos = 0;
		} else {
			linea_datos++;
		}
	}

	if (info->cantidad == SIN_POKEMON) {
		free(info->pokemones);
		free(info);
		fclose(archivo);
		return NULL;
	}

	fclose(archivo);
	return info;
}
```

La complejidad de la función `pokemon_buscar` es O(n) ya que se hace una serie de operaciones tomando en cuenta el tamaño de los datos. Especificamente la cantidad de pokemones.

```c
pokemon_t *pokemon_buscar(informacion_pokemon_t *ip, const char *nombre)
{
	if (ip == NULL || nombre == NULL) {
		return NULL;
	}
	int i = 0;
	bool encontrado = false;

	while (i < ip->cantidad && !encontrado) {
		if (strcmp(ip->pokemones[i].nombre, nombre) == 0) {
			encontrado = true;
		}
		i++;
	}

	if (encontrado) {
		return &(ip->pokemones[i - 1]);
	} else {
		return NULL;
	}
}
```

La función `con_cada_pokemon` contiene un algoritmo de ordenamiento Bubble sort el cual tiene una complejidad de O(n^2) y luego se hace una serie de operaciones que depende de la cantidad de pokemones por lo que su complejidad seria O(n). Asi que en definitiva la complejidad de la función `con_cada_pokemon` es de O(n^2) ya que ambas funciones no se encuentran al mismo nivel, por lo que tomo en cuenta la de mayor peso.

```c
void ordenar_pokemones(pokemon_t *pokemones, int cantidad_pokemones)
{
	if (pokemones == NULL) {
		return;
	}

	bool ordenado = false;
	int i = 0;
	pokemon_t aux;
	while (i < cantidad_pokemones - 1 && !ordenado) {
		ordenado = true;
		for (int j = 0; j < cantidad_pokemones - i - 1; j++) {
			if (strcmp(pokemones[j].nombre,
				   pokemones[j + 1].nombre) > 0) {
				aux = pokemones[j];
				pokemones[j] = pokemones[j + 1];
				pokemones[j + 1] = aux;
				ordenado = false;
			}
		}
		i++;
	}
	return;
}

int con_cada_pokemon(informacion_pokemon_t *ip, void (*f)(pokemon_t *, void *),
		     void *aux)
{
	if (ip == NULL || f == NULL) {
		return 0;
	}
	ordenar_pokemones(ip->pokemones, ip->cantidad);
	for (int i = 0; i < ip->cantidad; i++) {
		f(&(ip->pokemones[i]), aux);
	}
	return ip->cantidad;
}
```

La complejidad de la función `con_cada_ataque` es O(n) ya que se hace una serie de operaciones tomando en cuenta el tamaño de los datos. Especificamente la cantidad de ataques.

```c
int con_cada_ataque(pokemon_t *pokemon,
		    void (*f)(const struct ataque *, void *), void *aux)
{
	if (pokemon == NULL || f == NULL) {
		return 0;
	}

	for (int i = 0; i < pokemon->cantidad_ataques; i++) {
		f(&(pokemon->ataques[i]), aux);
	}
	return pokemon->cantidad_ataques;
}
```

La complejidad de la función `pokemon_destruir_todo` es O(1) ya que se hace una serie de operaciones sin importar el tamaño de los datos.

```c
void pokemon_destruir_todo(informacion_pokemon_t *ip)
{
	if (ip == NULL) {
		return;
	}
	free(ip->pokemones);
	free(ip);
	return;
}
```