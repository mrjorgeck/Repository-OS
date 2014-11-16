/*
Repository-OS
=============

Operative System
*/

#include <stdio.h>
#include <stdlib.h>
#include<ctype.h>


//constantes

#define quantum 5
#define t_pagina 15
#define cambio 75
#define dir_inicial 1000

//Variables para memoria

int existe,
	contador,
	procesos,
	estado_memoria,
	indice,
	id_ant[10],
	mp_ant[10],
	id_act[10]={-1,-1,-1,-1,-1,-1,-1,-1,-1,-1},
	n_p_act[10],
	dir_v_s[10],
	dir_v_d[10],
	dir_real[10],
	cont[10],
	mp[10]={-1,-1,-1,-1,-1,-1,-1,-1,-1,-1},
	situacion_cola[10][9];	



//----------------------------------------------------------------
//----estructura de nodo
//----------------------------------------------------------------


typedef struct nodo
{
	int id;
	int tamanio;
	int paginas;
	int p_actual;
	struct nodo *sig;
} nodot;
//----------------------------------------------------------------
//--Apuntadores para manejo de nodos

typedef nodot *ptrNodo;	//apuntador a tipo nodo
typedef nodot *ptrH;	//apuntador al primer nodo de la lista
typedef nodot *ptrT;	//apuntador al primer nodo de la lista


//----------------------------------------------------------------
//-----Funciones para manejo de nodos
//----------------------------------------------------------------

//----------------------------------------------------------------
//----- Modulo de buscar: compara id con el identificador almacenado en el nodo actual

ptrNodo buscaNodo(ptrH *h, ptrT *t, int id){
	ptrNodo nodo=*h;
	while(nodo!=NULL&&(nodo->id)!=id){
		nodo=nodo->sig;
	}
	return nodo;
}

ptrNodo buscaNodoSig(ptrH *h, ptrT *t, int id){
	ptrNodo nodo=*h;
	while(nodo!=NULL&&(nodo->id)!=id){
		nodo=nodo->sig;
	}
	if(nodo!=NULL)
	   {
	   	   if(nodo==*t)
	   	   	  nodo=*h;
	   	   	else
	   	   	  nodo=nodo->sig;
	   }
	return nodo;
}
//----------------------------------------------------------------
//----- Buscar
ptrNodo buscaNodoAnterior(ptrH *h, ptrT *t, int id){
	ptrNodo nodo=*h;
	existe=0;
	if(nodo->id==id){
		existe=1;
		return NULL;
		}
	else{
		while(nodo->sig!=NULL){
			if((nodo->sig->id)==id){
				existe=1;
				return nodo;
			}
			nodo=nodo->sig;
		}
	}
	return nodo;
}

//----------------------------------------------------------------
/*Crea un nuevo nodo y en el campo de ID almacena el valor como parametro
 regresa el nodo recien creado*/


int validaVacio(ptrH *h){
	return (*h==NULL ? 1:0);
}
 void listaNodo(ptrNodo nodo){
	printf("\nProceso: %d\t Tamanio: %d",nodo->id, nodo->tamanio);
 }

//----------------------------------------------------------------
//-----Lista los nodos almacenados
 void listar(ptrH *h){
	if(*h==NULL)
		printf("\n\tNo hay procesos");
	else{
		ptrNodo i=*h;
		printf("\n\n\tTabla de procesos\n");
		while(i!=NULL){
			printf("\nProceso: %d\t Tamanio: %d\t Paginas %d\n", i->id, i->tamanio, i->paginas);
			i=i->sig;
			}
		}
 }


//----------------------------------------------------------------
//----- Modulo de insertar: crea nodo

 ptrNodo creaNodo(int id, int tamanio){
 	int paginas;
	ptrNodo nuevoNodo=(ptrNodo)malloc(sizeof(nodot));
	paginas=tamanio/t_pagina;
	if(tamanio%t_pagina==0)
		paginas-=1;
	if(nuevoNodo!=NULL){
		nuevoNodo->id=id;
		nuevoNodo->tamanio=tamanio;
		nuevoNodo->paginas=paginas;
		nuevoNodo->p_actual=0;
		nuevoNodo->sig=NULL;
	}
	return nuevoNodo;
}

//----------------------------------------------------------------
//----- Modulo de insertar: inserta un nodo al inicio de la lista

void insertaNodoInicio(ptrH *h, ptrT *t, int id, int tamanio){
	ptrNodo nuevo=creaNodo(id, tamanio);
	if(*h==NULL)	//SÌ la lista est· vacÌa
		*t=nuevo;
	nuevo->sig=*h;
	*h=nuevo;
}


//----------------------------------------------------------------
//----- Modulo de insertar: inserta nodo al final de la lista

void insertaNodoFinal(ptrH *h, ptrT *t, int id, int tamanio){
	ptrNodo nuevo=buscaNodo(h, t, id);
	if (nuevo==NULL)
	{
	   nuevo=creaNodo(id, tamanio);
	   ptrNodo fin=*t;
	   if(*h==NULL)
		   *h=nuevo;
	   else
		   fin->sig=nuevo;
	   *t=nuevo;
	}
	else printf("El proceso %d ya existe, tiene un tamanio de %d", nuevo->id, nuevo->tamanio);
}

//----------------------------------------------------------------
//----- Eliminar nodo

void eliminaNodo(ptrH *h, ptrT *t, ptrNodo ant){
	if(ant->sig==*t){
		ant->sig=NULL;
		*t=ant;
		}
	else
		ant->sig=ant->sig->sig;
	
}

//----------------------------------------------------------------
//Insertar nodo
/*Agrega a la lista que regresa como parametro , e inserta otro nodo despues de el*/
void insertaNodo(ptrH *h,ptrT *t,ptrNodo ant,int id , int tamanio){
	
	
	ptrNodo nuevo=creaNodo(id, tamanio);
	ptrNodo aux=buscaNodo(h, t, id);
if (aux==NULL)
{
	ptrNodo fin=*t;
	if(ant!=NULL){	//si el nodo anterior existe
		if(ant==*t)	//si est· al final de la lista se inserta en el final
			insertaNodoFinal(h,t,id, tamanio);
		else{ //si no est· al final de la lista
			nuevo->sig=ant->sig;
			ant->sig=nuevo;
		}
	}
	else {	//si no hay nodo anterior
		*h=nuevo;
		*t=nuevo;
	}
}
else printf("Ya existe el proceso: %d y tiene un tamaño de %d", aux->id, aux->tamanio);
}




//----------------------------------------------------------------
//----- Modulo de insertar: inserta nodo a la derecha
void insertaNodoD(ptrH *h, ptrT *t,int id, int nuevo, int tamanio){
	ptrNodo ant=buscaNodo(h,t,id);
	if(ant!=NULL)
		insertaNodo(h,t,ant,nuevo, tamanio);
	else
		printf("No se encuentra proceso %d", id);
}

//----------------------------------------------------------------
//----- Eliminar nodo
ptrNodo eliminaNodoB(ptrH *h, ptrT *t, int id){
	ptrNodo ant=buscaNodoAnterior(h,t,id);
	ptrNodo aux=*h;
	if(existe==1){	//si encontro el proceso
		if(ant!=NULL){	//si no es el primer nodo
			aux= ant->sig;
			if(aux==*t)
				aux=*h;
			else
				aux=aux->sig;
				
			eliminaNodo(h,t,ant);
			}
		else if(*h==*t)
			aux=*h=*t=NULL;
			else
			{
			   aux=aux->sig;
			   *h=(aux); 
			   
			}
	}
	else
		printf("No se encuentra el Proceso: %d",id);
	return aux;
}
//----------------------------------------------------------------
//----- Actualizar valor almacenado en nodo

void actualizaDato(ptrH *h, ptrT *t, int id, int tamanio){
	ptrNodo aux=buscaNodo(h,t,id);
	if(aux!=NULL)
		aux->tamanio=tamanio;
	else
		printf("No se encuentra proceso: %d", id);
}


//----------------------------------------------------------------
//------Funciones auxiliares
//----------------------------------------------------------------

void muestraMemoria()
{
   int i,j;
   printf("\n\n\tEstado de memoria:\n");
   printf("\t Proceso\t Pagina\t\t Proceso\t Pagina\t\t\t\t\t\t Inst\t\n");
   printf("\t Act\t\t Act\t\t Ant\t\t Ant\t\t Dir V\t\t Dir R\t\t ejec\t cola");
   for(i=0; i<10; i++)
   {
   	   printf("\n\n\t %d\t %d\t\t %d\t\t %d\t\t %d\t\t %d %d\t\t %d\t\t %d\t", n_p_act[i],id_act[i], mp[indice] ,id_ant[i], mp_ant[i], dir_v_s[i], dir_v_d[i], dir_real[i], cont[i]);
   	   j=0;
   	   while(j<9&&situacion_cola[i][j]!=0)
   	   {
   	   		printf(" %d", situacion_cola[i][j]);
   	   		j++;
   	   }
   }
}







//----------------------------------------------------------------
//------Funcion principal
//----------------------------------------------------------------

int main(
			int argc, // Tamaño del arreglo
 			char *argv[]     // arreglo que tiene los comandos ingresados
 		)
{
	//para manejo de nodos
	int datoN;
	int op;
	ptrH h=NULL;
	ptrT t=NULL;
	ptrNodo aux=NULL;
	//para lectura de archivo
	int i,
		id,
		tamanio,
		j=0;

    FILE *archivo;
    char  	caracter,
    		cadena[10];

    //manejo de programa
    int bandera2=0,
    	terminap=0,
    	bandera3=0,					//indica que hay salto de memoria
    	paginasTotal=0,
    	paginasActual=0;
	

	    
	    
	    
	
	
	//Obtener procesos y tamaño	
	//---------------------------------------------------------------------------



    
    //Muestra nombre del archivo
    
    printf( "\nAccediendo al archivo: " );
    
    for( i = 1; i < argc; i++ ){
        printf( "%s\n",argv[i]);
	archivo=fopen(argv[i],"r");
	}
	
	//abre archivo
	if(archivo==NULL)
		printf("No se puede abrir el archivo\n");
	else{
		printf("EL archivo se abrio correctamente\n");
	while(feof(archivo)==0){
	
	
	caracter=fgetc(archivo);							//Obtiene contenido de archivo
		
		if(caracter==' '|| iscntrl(caracter)){			//si encuentra espacio o dato de control (fin de linea)
			if (caracter == ' '){						//espacio
	   			id = atoi(cadena);			//convierte el contenido de cadena en un entero y lo guarda en id
				
			}
			else {
				if(iscntrl(caracter))					//Dato de control (fin de linea)
			{
	   			tamanio = atoi(cadena);		//convierte el contenido de cadena en un numero y lo guarda como tamanio
	  			insertaNodoFinal(&h,&t,id, tamanio);	//crea nuevo nodo de proceso
	  			procesos++;
	  		}
	  		}
			for (i=0; i<=j; i++)						//limpia cadena
				cadena[i]=0;
			j=0;								//inicia cadena de nuevo al inicio
		}
		else{ 								//Lee el numero id o tamaño y lo guarda en un arreglo
	   	   cadena[j]=caracter;
	   	   j++;
		}
	}
	}
	fclose(archivo);									//Cierra el archivo	





//---------------------------------------------------------------------------
//Round Robin



	aux=h;

	if(aux!=NULL)
	{
//Valida la estradas de procesos y obtiene numero total de paginas	
	

		do{
			while((aux->tamanio)<=0||(aux->id<0)){			//Si el proceso no tiene un tamaño valido, sale de lista de procesos
	    	 	printf("\nDato invalido");
	    	 	aux=eliminaNodoB(&h,&t,aux->id);			//Elimina proceso y obtiene el siguiente
	    	 	procesos--;									//Reduce la cantidad de procesos activos
	    	 }	
			paginasTotal+=aux->paginas;					//Obtiene numero total de páginas por proceso válido
			aux=buscaNodoSig(&h,&t,aux->id);
		} while(aux!=h);
		
		paginasActual=paginasTotal;

		listar(&h);

		while(contador<cambio && h!=NULL)
		{
			int bandera=1,
				max;									//Toma el valor del quantum o de la cantidad de procesos restantes para cumplir 75 inst. si sin menos



			estado_memoria=1;
			for (i=0; i<10; i++)
				if(mp[i]==-1)
					estado_memoria=0;

			//Se asegura de que esté llena la memoria con procesos validos
			if(indice>=procesos && estado_memoria==0 && paginasTotal>0)   		//Si no se ha llenado la memoria y aun hay paginas libres
				do
				{
					if(aux->p_actual<aux->paginas) 				//Si le quedan paginas al proceso
					{
						aux->p_actual++;						//Incrementa el valor de la pagina actual
						paginasTotal--;							//Decrementa el valor total de las paginas
						n_p_act[indice]=aux->p_actual;			//Nueva pagina actual
						bandera=1;								//Sale de condicion
					}
					else
					{
						printf("\tno proc %d", aux->id);
						aux=buscaNodoSig(&h,&t, aux->id);			//Cambia al proceso siguiente en lista
						bandera=0;									//Permite el ciclo, para poder verificar de nuevo
						
					}

				}	while(bandera==0);								//Mientras no se encuentre 
			printf("paginasT %d", paginasTotal);

			//-------------------------------
			//inicia obtencion de valores

			mp[indice]=indice;									//Numero de marco de pagina
			
			//asigna valores a marco de pagina anterior y proceso anterior
			if(bandera3==0){									//si hubo un salto de pagina antes
				if(indice==0)
				{
					mp_ant[indice]=mp[9];
					id_ant[indice]=id_act[9];
				}
				
				else
				{
					id_ant[indice]=id_act[indice-1];
					mp_ant[indice]=mp[indice-1];
				}
			}
			else
				bandera3=0;




			id_act[indice]=aux->id;								//Indica proceso actual
			

			//Indica memoria virtual
			if(aux->tamanio<=quantum)							//Proceso termina antes que quantum
			{
				dir_v_d[indice]+=aux->tamanio;
				contador+=aux->tamanio;
				terminap=1;										//Bandera que indica que un proceso termino
				dir_real[indice]=-1;							//Indica que no tiene direccion real
			}
			else												//El quantum termina antes que el proceso
			{
				dir_v_d[indice]+=quantum;							
				contador+=quantum;
				actualizaDato(&h,&t, aux->id, aux->tamanio - quantum);	
			
				if (dir_v_d[indice]>=t_pagina)					//Cambia valor de pagina
				{
					dir_v_s[indice]+=1;
					dir_v_d[indice]=0;
					aux->p_actual++; 
					paginasTotal--;
				}
			    dir_real[indice]=dir_inicial + (mp[indice] * t_pagina) + dir_v_d[indice];	//Calcula direccion real
			}
			
			
			if(terminap==1)
			{
				mp[indice]=-1;
				dir_real[indice]=-1;
			}

			
		//imprime datos de prueba
		printf("\n\t%d\t%d\t%d\t%d\t%d\t%d , %d\t%d\t%d",n_p_act[indice],id_act[indice],mp[indice] , id_ant[indice],mp_ant[indice],dir_v_s[indice], dir_v_d[indice], dir_real[indice], contador);
		
		if(terminap==1)										//Si termino proceso
			{
				printf("\tTermina proceso %d", aux->id);
				aux=eliminaNodoB(&h,&t, id_act[indice]);		//Elimina proceso
				id_ant[indice]=-1;
				mp_ant[indice]=-1;
				id_act[indice]=-1;
				if (indice==0)
					indice=10;
				indice--;										//Vuelve a ocupar el mismo marco de memoria para otro proceso
				terminap=0;
				bandera3=1;

			}

		//cambio de indice
		if(paginasTotal>0)							//Aun quedan paginas para cargar			
		{
			
			if(indice==9)
			{	
				bandera2=1;							//por lo menos se lleno una ves la memoria
				indice=-1;
			}									
			indice++;			
		}
		else													//Ya no hay paginas para cargar
		{	int ultimo_tiro=indice;
			do{
				if(indice==9)
					indice=-1;
				indice++;

			}
			while(h!=NULL && id_act[indice]==-1 && paginasTotal<=0);			
			id_ant[indice]=id_act[ultimo_tiro];
			mp_ant[indice]=mp[ultimo_tiro];
			bandera3=1;

		}

		
		//Previene nodo a trabajar
		if(h!=NULL && bandera2==1)
			aux=buscaNodo(&t,&h,id_act[indice]);
		else
			aux=buscaNodoSig(&h,&t, aux->id);
		
		

		}
	}

	
	
	//---------------------------------------------------------------------------


printf("\n\n");
listar(&h);

}
