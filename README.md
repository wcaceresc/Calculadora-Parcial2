#include <stdint.h>
#include "stm32l053xx.h"

void init();
void ky_filas(); //ESTA FUNCION ES PARA RECORRER LAS FILAS
void ky_col();   //ESTA FUNCION ES PARA RECORRER LAS COLUMNAS
void disp7(uint8_t); //ESTA FUNCION GUARDARA EL NUMERO PARA LUEGO RECORRER Y MOSTRAR EL NUMERO CORRESPONDIENTE
void delayMs(int n);

//uint16_t last_number = 0;
// DEFINICION DE LAS SUBMASCARAS SE ME HACE MAS FACIL VERLO CON LETRAS
#define FILA_1 (0x20)
#define FILA_2 (0x40)
#define FILA_3 (0x100)
#define FILA_4 (0x200)

uint8_t val = 0; //VARIABLE QUE GUARDARA VALORES TEMPORALES EN LOS RECORRIDOS

/////ESTAS VARIABLES TEMPORALES GUARDARAN LOS DATOS PARA LAS OPERACIONES MATEMATICAS
uint8_t digito1 = 0;
uint8_t digito2 = 0;
uint8_t suma = 0;
uint8_t temp1 = 0;
uint8_t temp2 = 0;

void init();

       //Variable temporal para guardar numero presionado
       uint8_t btn_press = 0;

int main(void)
 {
   init();

   /////ESTE CICLO SIRVE PARA QUE ESTE EJECUTANDO LAS FUNCIONES DE FILAS Y COLUMNAS QUE LAS RECORRA
   while(1)
       {
   	     ky_filas();
	     ky_col();
	   }
 }

void init()

   {
    //HABILITA LOS CLOCKS DE A,B Y C RESPECTIVAMENTE
	  RCC->IOPENR |=0x01<<0;
	  RCC->IOPENR |=0x01<<1;
	  RCC->IOPENR |=0x01<<2;

    //PUERTOS QUE SE VAN A USAR PARA CADA SEGMENTO
       GPIOB->MODER &=~(1<<17);	//PB8 A
	   GPIOB->MODER &=~(1<<5);	//PB2 B
	   GPIOB->MODER &=~(1<<7);	//PB3 C
	   GPIOB->MODER &=~(1<<9);	//PB4 D
	   GPIOB->MODER &=~(1<<13);	//PB6 E
	   GPIOB->MODER &=~(1<<15);	//PB7 F
	   GPIOB->MODER &=~(1<<19);	//PB9 G


   // COMUN Y LIMPIA EL PUERTO PA1 QUE FUE EL QUE USE
	   GPIOA->MODER &=~(1<<3);
	   GPIOA->BSRR = (0x03<<16);

}
void ky_filas(){

	//Filas como IN pull up PC5, PC6, PC7, PC8.

	GPIOC->MODER &=~(1<<10); //LIMPIA EL PIN PC5 PARA LUEGO PODERLOS ACTIVAR
	GPIOC->MODER &=~(1<<11); //ESTE FUE ELEGIDO PARA USAR CON LA RESISTENCIA PULL UP
	GPIOC->PUPDR |=(1<<10);

	GPIOC->MODER &=~(1<<12); //LIMPIA EL PIN PC6 PARA LUEGO PODERLOS ACTIVAR
	GPIOC->MODER &=~(1<<13); //ESTE FUE ELEGIDO PARA USAR CON LA RESISTENCIA PULL UP
	GPIOC->PUPDR |=(1<<12);

	GPIOC->MODER &=~(1<<16);//LIMPIA EL PIN PC7 PARA LUEGO PODERLOS ACTIVAR
	GPIOC->MODER &=~(1<<17);//ESTE FUE ELEGIDO PARA USAR CON LA RESISTENCIA PULL UP
	GPIOC->PUPDR |=(1<<16);

	GPIOC->MODER &=~(1<<18);//LIMPIA EL PIN PC8 PARA LUEGO PODERLOS ACTIVAR
	GPIOC->MODER &=~(1<<19);//ESTE FUE ELEGIDO PARA USAR CON LA RESISTENCIA PULL UP
	GPIOC->PUPDR |=(1<<18);

	//Columnas como OUT PB11,PC7,PA9,PC4.

	GPIOB->MODER &=~(1<<23);//LIMPIA PB11
	GPIOC->MODER &=~(1<<15);//LIMPIA PC7
	GPIOA->MODER &=~(1<<19);//LIMPIA PA9
	GPIOC->MODER &=~(1<<9); //LIMPIA PC4
}

void ky_col() // Recorrera las columans 1,2,3,4.

     {

	/*EXPLICARE ESTA PARTE QUE ES DONDE SE DESARROLLA LO MAS IMPORTANTE QUE SON LAS OPERACIONES MATEMATICAS Y DE QUE FORMA
	  UTILIZO CADA LINEA YA QUE ES DONDE PUEDE CREAR CONFUSION O NO ESTAR CLARO COMO FUNCIONA. POR LO TANTO AQUI LO EXPLICO:

	  PASO 1 PRESIONO UNA TECLA CUALQUIERA,  ESTA SE GUARDA EN LA VARIABLE DIGITO 1 QUE PASARA A SER IGUALDAD A LA VARIABLE TEMP1
	  PASO 2 DEBO FORZOSAMENTE PRESIONAR ATERISCO PARA QUE LIMPIE EL DISPLAY Y QUEDE GUADADA EN LA VARIABLE EL NUMERO QUE SE PRESIONO
	  PASO 3 EL PROGRAMA VUELVE A ENTRAR A LOS CICLOS, SE PRESIONA UNA SEGUNDA TECLA QUE SERA GUARDADA EN LA VARIABLE DIGIT2
	  PASO 4 DEBO PRESIONAR FORZOSAMENTE NUMERAL PARA QUE, ENTONCES GUARDE DICHO NUMERO EN LA VARIABLE DIGIT2 QUE SERA IGUALADA A TEMP2
	  PASO 5 SE DEBE APACHAR EL OPERADOR DE SU ELECCCION SEA SUMA RESTA MULTIPLICACION O DIVISION
	  PASO 6 DEPENDIENDO CUAL SE APACHE EJEMPLO: SI ES SUMA QUE CORRESPONDE A LA LETRA "A" ENTRA EN ESE "IF" YA QUE SE CUMPLE LA CONDICION
	         Y AHI MANDA A LLAMAR A LAS VARIABLES QUE TIENEN LOS VALORES GUARDADOS PREVIAMENTE, TAMBIEN TIENE CONDICIONES ANIDADAS
	         ES DECIR QUE DESPLIEGUE "E" DE ERROR CUANDO EL NUMERO SEA MAYOR QUE NUEVE YA QUE EN ESTE CASO SOLO USAMOS UN DISPLAY
	         ASI TAMBIEN, EN EL CASO DE LA DIVISION CUANDO SE DIVIDE ENTRE CERO, Y LA RESTA LE INCLUI QUE DESPLEGARA EL SEGMEN DE
	         EN MEDIO ANTES DEL NUMERO PARA QUE SE INDIQUE SI EL RESULTADO DE LA RESTA ERA NEGATIVO.

	 SEGURAMENTE NO ES UNA LOGICA DE LAS MAS FINAS ES UN POCO ENREDADA PERO LO HICE DE ESA MANERA PORQUE ME AHORRABA MUCHISIMAS LINEAS
	 DE CODIGO Y UN PAR DE CICLOS POR ESO DECIDI HACERLO DE ESA MANERA Y ES MUY FUCNCIONAL PORQUE SE PUEDE REPETIR LAS OPERACIONES
	 UNICAMENTE PRESIONANDO ASTERISCO O NUMERAL YA QUE POR LA ESTRUCTURA NOS PERMITE HACER INFINITAS OPERACIONES SIN QUE SE TENGA QUE
	 PRESIONAR MAS TECLAS YA QUE LAS LETRAS A,B,C,D QUE SON LOS OPERADORES, TAMBIEN FUNCIONAN COMO UN "ENTER" O "IGUAL" EN UNA
	 CALCULADORA REAL. UNICAMENTE UTILICE A MI FAVOR EL CICLO GLOBAL DEL CODIGO PARA QUE SE PUEDA REPETIR EL PROCESO CUANTAS VECES SE
	 DESEE*/


	//AQUI ABAJO COMENTARE CADA LINEA QUE DEMUESTRA LO DESCRITO EN ESTE BLOQUE DE EXPLICACION




	//RECORRE LA COLUMNA 1 ES DECIR LOS CARACTERRES 1-4-7 y *
	      GPIOB->ODR &=~(1<<11);
	      GPIOC->ODR |=(1<<7);
	      GPIOA->ODR |=(1<<9);
	      GPIOC->ODR |=(1<<4);
	      delayMs(5);

	         if(!(GPIOC->IDR & FILA_1)) //ESTE NIEGA LA ENTRADA PARA QUE FUNCIONE EL PULL UP Y SI ES LA FILA 1
	        	                        //REVISA QUE SEA APACHADO EL UNO POR LO TANTO LA VARIABLE disp7 TOMA EL VALOR
	        	                        //0X01 QUE POSTERIORMENTE ENTRARA AL LA MAQUINA DE ESTADOS (SWITCH CASE) PARA
	        	                        //QUE LE DIGA AL DISPLAY QUE ES LO QUE VA MOSTRA CUALES SEGMENTOS ENCENDER
	         {
	        	disp7(0x01);
            	digito1=1;  //ESTA GUARDA EL PRIMER VALOR PRESIONADO
                digito2=1;  //ESTA GUARDA EL SEGUNDO VALOR PRESIONADO



	         }
	             else if(!(GPIOC->IDR & FILA_2))
	                   {

	            	    disp7(0x04);
		                digito1=4;//ESTA GUARDA EL PRIMER VALOR PRESIONADO
                        digito2=4;//ESTA GUARDA EL SEGUNDO VALOR PRESIONADO
	                   }
	                   else if(!(GPIOC->IDR & FILA_3))
	                         {

	                	      disp7(0x07);
		                      digito1=7;//ESTA GUARDA EL PRIMER VALOR PRESIONADO
                              digito2=7;//ESTA GUARDA EL SEGUNDO VALOR PRESIONADO

	                         }
	                         else if(!(GPIOC->IDR & FILA_4))
	                         {



	                        /////////////OJO OJO OJO A ESTA PARTE ////////////////

	                         temp1=digito1; //AQUI ES DONDE ASTERISCO GUARDA EL PRIMER DIGITO PRESIONADO
	                          disp7(0xFF);   // Y  EN VEZ DE MOSTRAR ALGO SOLO LO LIMPIA
	                         }
    //COL2 (2-5-8-0)
	GPIOB->ODR |=(1<<11);
	GPIOC->ODR &=~(1<<7);
	GPIOA->ODR |=(1<<9);
	GPIOC->ODR |=(1<<4);
	delayMs(5);

	          if(!(GPIOC->IDR & FILA_1))
	            {
            	 disp7(0x02);
		         digito1=2;//ESTA GUARDA EL PRIMER VALOR PRESIONADO
	             digito2=2;//ESTA GUARDA EL SEGUNDO VALOR PRESIONADO
                }
	             else if(!(GPIOC->IDR & FILA_2))
	                    {
		  			     disp7(0x05);
    	                 digito1=5;//ESTA GUARDA EL PRIMER VALOR PRESIONADO
		                 digito2=5;//ESTA GUARDA EL SEGUNDO VALOR PRESIONADO

		                }
	                    else if(!(GPIOC->IDR & FILA_3))
	                           {
		   		                disp7(0x08);
    	                        digito1=8;//ESTA GUARDA EL PRIMER VALOR PRESIONADO
		                        digito2=8;//ESTA GUARDA EL SEGUNDO VALOR PRESIONADO

			                   }
	                            else if(!(GPIOC->IDR & FILA_4))
	                                   {
                                        disp7(0x00);
		                                digito1=0;//ESTA GUARDA EL PRIMER VALOR PRESIONADO
		                                digito2=0;//ESTA GUARDA EL SEGUNDO VALOR PRESIONADO

	                                   }
    //COL3 (3-6-9-#)
	GPIOB->ODR |=(1<<11);
	GPIOC->ODR |=(1<<7);
	GPIOA->ODR &=~(1<<9);
	GPIOC->ODR |=(1<<4);
	delayMs(5);

	if(!(GPIOC->IDR & FILA_1))
	  {
		disp7(0x03);
        digito1=3;//ESTA GUARDA EL PRIMER VALOR PRESIONADO
        digito2=3;//ESTA GUARDA EL SEGUNDO VALOR PRESIONADO

	  }
	       else if(!(GPIOC->IDR & FILA_2))
	              {
                   disp7(0x06);
	               digito1=6;//ESTA GUARDA EL PRIMER VALOR PRESIONADO
	               digito2=6;//ESTA GUARDA EL SEGUNDO VALOR PRESIONADO

			      }
	              else if(!(GPIOC->IDR & FILA_3))
	                     {
	            		  disp7(0x09);
		                  digito1=9;//ESTA GUARDA EL PRIMER VALOR PRESIONADO
		                  digito2=9;//ESTA GUARDA EL SEGUNDO VALOR PRESIONADO

		                 }

	            //////////  OJO  OJO  OJO A ESTA PARTE   //////////////////////

	                        else if(!(GPIOC->IDR & FILA_4))
	                            {
	                    	     temp2=digito2;  //AL PRESIONAR AQUI EL NUMERAL GUARDARA EL SEGUNDO DIGITO
	                    	     disp7(0xFF); // LO LIMPIA Y NO MUESTRA NADA EN EL DISPLAY
		                   	    }
    //COL4 (A = + ,B = - ,C = * ,D = /)
	GPIOB->ODR |=(1<<11);
	GPIOC->ODR |=(1<<7);
	GPIOA->ODR |=(1<<9);
	GPIOC->ODR &=~(1<<4);
	delayMs(5);
///////////////////////// SUMA   EXPLICACION  //////////////////////////////////////////////////////////////////////////////////////







	             if(!(GPIOC->IDR & FILA_1))
                         {
                          suma=temp1+temp2; //TOMA LOS DOS VALORES ALMACENADOS Y LOS SUMA

                           if (suma <= 9) //SI ES MENOR O IGUAL A 9
	                	  {
	                		disp7(suma); // MOSTRARA LA SUMA COMO TAL DE LAS DOS VARIABLES
	                		GPIOA->ODR |=(1<<1);
	                		delayMs(5);
	                	  }
	                	  else {
	                		  disp7(0x0E); //DE LO CONTRARIO SI ES MAYOR QUE MUESTRE ERROR "E"
	                		  delayMs(5);
	                	       }


      /////////////////////////   RESTA EXPLICACION   /////////////////////////////////////////////////////////////////////////////////////
		                 }
	                      else if(!(GPIOC->IDR & FILA_2)) //AL PRESIONAR LETRA B
	                             {
                                  suma=(temp1-temp2); //RESTA LOS DOS VALORES DE LAS VARIABLES

                                  if (temp2 > temp1) //PERO SI EL SEGUNDO VALOR ES MAYOR QUE EL PRIMERO
                                  {
                                	  suma=suma*-1; /////QUE HAGA LA SUMA NORMAL PERO QUE LE PONGA EL SIGNO MENOS
                                	  disp7(0x0B); //ESTE ES EL CASO PARA MOSTRAR EL SIGNO MENOS
                                	  disp7(suma); //LUEGO QUE SE PASE AL CASO PARA MOSTRAR EL NUMERO


                                  }
                                  else if (temp2 < temp1)
                                  {
                                	  disp7(suma); // Y AQUI SI ES UNA RESTA CON EL PRIMER DIGITO MAYOR QUE EL SEGUNDO QUE MUESTRE
                                	               // LA RESTA NORMAL SIN EL SIGNO NEGATIVO

                                  }



	                             }

////////////////////////   MULTIPLICACION  EXPLICACION //////////////////////////////////////////////////////////////////////////////
	                          	 else if(!(GPIOC->IDR & FILA_3))
	                                    {
                                         suma=temp1*temp2;//MULTIPLICA LAS DOS VARIABLES DE LOS VALORES GUARDADOS
                                         if (suma <= 9)//SI ES QUE EL RESULTADO SEA MENOR O IGUAL QUE NUEVE
                                         {
		                           		 disp7(suma); //DESPLIEGUE EL DIGITO
                                         GPIOA->ODR |=(1<<1);
		                                 delayMs(5);
                                         }
                                         else {
                                         disp7(0x0E); //SI ES MAYOR ANUEVE QUE SAQUE ERROR
                                         delayMs(5);
                                         }
		                              	}

////////////////////////   DIVISION EXPLICACION /////////////////////////////////////////////////////////////////////////////////
	                                    else if(!(GPIOC->IDR & FILA_4))
	                                           {
	                                    	    suma=temp1/temp2;//OPERACION DE DIVICION DE LAS VARIABLES

	                                    	    if (temp2 == 0)
	                                    	    {

	                                    	      disp7(0x0F); //SI ES CERO
	                                    	    }

	                                    	    else if(suma > 0 )
	                                    	    {
	                                    	    disp7(suma); //SI ES MAYOR A CERO
	                                    	    }

	                                    	    else if(suma>1)
	                                    	    {

	                                    	    disp7(0x0C); //SI SE DIVIDE DE FORMA INCORRECTA QUE USE ESTE CASO
		                                        GPIOA->ODR &=~(1<<1);
		                                        delayMs(5);
	                                    	    }
	                                    	  }
} //Fin de ky_col


//////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

void disp7(uint8_t val)
{
	   btn_press = val;
	   GPIOB->BSRR = (0x3DC << 16); // RESETEA TODOS LOS BITS A 0

	//Maquina de estados para el despliegue de los numeros o letras
	     switch (val)
	        {

   // AQUI ENTRA A LOS CASOS PARA DESPLEGAR EN EL DISPLAY LOS SEGMENTOS A ENCENDER
	// EL CASO C EL MAS PEQUEÑO SOLO MUESTRA EL GUIN MEDIO PARA INDICAR QUE ES NEGATIVO


	        case 0x00:
	            GPIOB->ODR |= 0x01 << 8;
	            GPIOB->ODR |= 0x01 << 2;
	            GPIOB->ODR |= 0x01 << 3;
	            GPIOB->ODR |= 0x01 << 4;
	            GPIOB->ODR |= 0x01 << 6;
	            GPIOB->ODR |= 0x01 << 7;
	            break;
	        case 0x01:
	            GPIOB->ODR |= 0x01 << 2;
	            GPIOB->ODR |= 0x01 << 3;
	            break;
	        case 0x02:
	            GPIOB->ODR |= 0x01 << 8;
	            GPIOB->ODR |= 0x01 << 2;
	            GPIOB->ODR |= 0x01 << 9;
	            GPIOB->ODR |= 0x01 << 6;
	            GPIOB->ODR |= 0x01 << 4;
	            break;
	        case 0x03:
	            GPIOB->ODR |= 0x01 << 8;
	            GPIOB->ODR |= 0x01 << 2;
	            GPIOB->ODR |= 0x01 << 9;
	            GPIOB->ODR |= 0x01 << 3;
	            GPIOB->ODR |= 0x01 << 4;
	            break;
	        case 0x04:
	            GPIOB->ODR |= 0x01 << 7;
	            GPIOB->ODR |= 0x01 << 9;
	            GPIOB->ODR |= 0x01 << 2;
	            GPIOB->ODR |= 0x01 << 3;
	            break;
	        case 0x05:
	            GPIOB->ODR |= 0x01 << 8;
	            GPIOB->ODR |= 0x01 << 7;
	            GPIOB->ODR |= 0x01 << 9;
	            GPIOB->ODR |= 0x01 << 3;
	            GPIOB->ODR |= 0x01 << 4;
	            break;
	        case 0x06:
	        	GPIOB->ODR |= 0x01 << 8;
	        	GPIOB->ODR |= 0x01 << 7;
	            GPIOB->ODR |= 0x01 << 6;
	            GPIOB->ODR |= 0x01 << 4;
	            GPIOB->ODR |= 0x01 << 3;
	            GPIOB->ODR |= 0x01 << 9;
	            break;
	        case 0x07:
	            GPIOB->ODR |= 0x01 << 8;
	            GPIOB->ODR |= 0x01 << 2;
	            GPIOB->ODR |= 0x01 << 3;
	            break;
	        case 0x08:
	            GPIOB->ODR |= 0x01 << 8;
	            GPIOB->ODR |= 0x01 << 2;
	            GPIOB->ODR |= 0x01 << 3;
	            GPIOB->ODR |= 0x01 << 4;
	            GPIOB->ODR |= 0x01 << 6;
	            GPIOB->ODR |= 0x01 << 7;
	            GPIOB->ODR |= 0x01 << 9;
	            break;
	        case 0x09:
	            GPIOB->ODR |= 0x01 << 9;
	            GPIOB->ODR |= 0x01 << 7;
	            GPIOB->ODR |= 0x01 << 8;
	            GPIOB->ODR |= 0x01 << 2;
	            GPIOB->ODR |= 0x01 << 3;
	            break;

	        case 0x0E:
	            GPIOB->ODR |= 0x01 << 8;
	            GPIOB->ODR |= 0x01 << 4;
	            GPIOB->ODR |= 0x01 << 7;
	            GPIOB->ODR |= 0x01 << 6;
	            GPIOB->ODR |= 0x01 << 9;
	            break;

	        case 0x0C:

	       	    GPIOB->ODR |= 0x01 << 9;  //ESTE CASO USA EL SEGMENTO DE EN MEDIO CUANDO LA RESTA ES NEGATIVA
	       	    break;



	       	    /// AQUI DESPLIEGA UNA PALABARA QUE LA PUSE DE MAS UN ERROR SOLO ERA PARA EJEMPLIFICAR COMO
	       	    // TAMBIEN PODEMOS ESCRIBIR PALABRAS DE UNA FORMA SENCILLA (NO FUNCIONAL) PERO SENCILLA
	       	    //SOLO FUE A MODO DE EJEMPLO MIS DICULPAS SI ESTA DE MAS
	        case 0x0F:

	            GPIOB->ODR |= 0x01 << 3;
	            GPIOB->ODR |= 0x01 << 4;
	            GPIOB->ODR |= 0x01 << 6;
	            GPIOB->ODR |= 0x01 << 7;
	            GPIOB->ODR |= 0x01 << 9;
	            delayMs(500);

	            GPIOB->ODR &= ~0x01 << 3;
	            GPIOB->ODR &= ~0x01 << 4;
	            GPIOB->ODR &= ~0x01 << 6;
	            GPIOB->ODR &= ~0x01 << 7;
	            GPIOB->ODR &= ~0x01 << 9;
	            delayMs(5);


	            GPIOB->ODR |= 0x01 << 3;
	            GPIOB->ODR |= 0x01 << 4;
	            GPIOB->ODR |= 0x01 << 6;
	            GPIOB->ODR |= 0x01 << 9;
	            delayMs(500);

	            GPIOB->ODR &= ~0x01 << 3;
	            GPIOB->ODR &= ~0x01 << 4;
	            GPIOB->ODR &= ~0x01 << 6;
	            GPIOB->ODR &= ~0x01 << 9;
	            delayMs(5);


	            GPIOB->ODR |= 0x01 << 3;
	            GPIOB->ODR |= 0x01 << 4;
	            GPIOB->ODR |= 0x01 << 6;
	            GPIOB->ODR |= 0x01 << 7;
	            GPIOB->ODR |= 0x01 << 9;
	            delayMs(500);

	            GPIOB->ODR &= ~0x01 << 3;
	            GPIOB->ODR &= ~0x01 << 4;
	            GPIOB->ODR &= ~0x01 << 6;
	            GPIOB->ODR &= ~0x01 << 7;
	            GPIOB->ODR &= ~0x01 << 9;
	            delayMs(5);

	            GPIOB->ODR |= 0x01 << 3;
	            GPIOB->ODR |= 0x01 << 4;
	            GPIOB->ODR |= 0x01 << 6;
	            GPIOB->ODR |= 0x01 << 9;
	            delayMs(500);

	            GPIOB->ODR &= ~0x01 << 3;
	            GPIOB->ODR &= ~0x01 << 4;
	            GPIOB->ODR &= ~0x01 << 6;
	            GPIOB->ODR &= ~0x01 << 9;
	            delayMs(500);


	            GPIOB->ODR |= 0x01 << 8;
	            GPIOB->ODR |= 0x01 << 4;
	            GPIOB->ODR |= 0x01 << 7;
	            GPIOB->ODR |= 0x01 << 6;
	            GPIOB->ODR |= 0x01 << 9;
	            break;

	        case 0x0B:
        	 	GPIOB->ODR |= 0x01 << 9;
        	 	delayMs(500);
  	        	break;

	        }
	}

	void delayMs(int n)

	   {
		int i = 0;
		 while (n > 0)
		   {
		    n--;
		    i = 0;
		     while (i < 240)
		       {
		        i++;
	      	   }
		   }
	   }

	//TERMINA EL PROGRAMA ESPERO ESTE MEJOR EXPLICADO, ENTENDIBLE Y JUSTIFICADO EL PORQUE LO HICE DE DICHA MANERA
![image](https://github.com/wcaceresc/Calculadora-Parcial2/assets/161264041/3f6c818f-4f74-4c74-bce9-4f8dccccdbb3)
