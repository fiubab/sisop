Vamos a crear una shell!
pasos:
	1 esperamos comando
	2 parseamos
	3 ejecutamos - poder hacerlo en 2do plano
	4 volvemos a esperar
	5 cerramos ante una SIGNAL
	

---


Que son redirecciones?
	![[pipe.png]]
	
	- redireccionar:
		-- stdin->fd=0/stdout->fd=1 a un arhivo
		-- pipes (entrada/salida)
		
	- usaremos dup2, open y close
		dup -> duplica un puntero (fd)	- vamos a querer pasar el stdout a un 
		archivo
		
	- dup2(oldfd, newfd)
		oldfd --> a donde quiero apuntar
		newfd --> el fd q quiero q apunte ahi
		
	- osea vamos a crear un fd que apunte al archivo y
	usar esta llamada dup2(fd, 1)
	
	- Esto nos sirve para poder enviar el contenido del
	archivo mediante un pipe
	recursion
	
Ahora como usamos pipes dobles?
	![[pipe doble.png|345]]
		
	entiendo que es un poco de recursion
	cada proceso debe ser un proceso simple que con 
	1 pipe entre proceso	
	
Variables de entorno

	- Diccionario al que acceden todos los procesos
	- Como le pido a la shell que quiero usar una d
	estas?
	- El parser reconoce el $ y reemplaza por el valor
	de la variable
	- export crea variables de entorno para la shell en 
	ejecucion
	- si creamos variables sin el export, nos sirven
	para ejecutar algun comando con esas variables y se
	crean solamente en el scope de ese comando
	
Comandos built-in:

	- son comandos que no necesitan hacer un fork
	ni usar pipes. Se ejecutan en el proceso de la 
	shell