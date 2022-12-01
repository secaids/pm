### <a href="https://github.com/secaids/pm#exp---1---01-alp-for-8086-1">01-ALP-FOR-8086</a>
### <a href="https://github.com/secaids/pm#exp-02-interfacing-a-digital-output-led">02-Interfacing-a-Digital-output-LED</a>
### <a href="https://github.com/secaids/pm#exp--03-interfacing-a-digital-input-push-button">03-Interfacing-a-Digital-INPUT-push-button</a>
### <a href="https://github.com/secaids/pm/#exp-4-interfacing-a-16x2-type-lcd-display">04-Interfacing a 16X2 type LCD display</a>
### <a href="https://github.com/secaids/pm#exp-05-interfacing-a-4x4-keypad">05-Interfacing a 4X4 keypad</a>
### <a href="https://github.com/secaids/pm#exp-6-converting-analog-to-digital-signals">06-converting analog to digital signals</a>
### <a href="https://github.com/secaids/pm#ex-07-interfacing-lm35-temperature-sensor">07-Interfacing LM35 Temperature sensor</a>
### <a href="https://github.com/secaids/pm#exp-08-interfacing-seven-segment-display">08-Interfacing seven segment display</a>
### <a href="https://github.com/secaids/pm#exp-09-configuring-uart-for-seral-data-transmission">09-Configuring UART for seral data transmission</a>

## <a href="https://github.com/ShafeeqAhamedS/EXPERIMENT--01-ALP-FOR-8086.git">Exp - 1 - 01-ALP-FOR-8086</a>
### sum - 8 bit
```
MOV SI,1200h
MOV CL,00h
MOV AL,[SI]
MOV BL,[SI+1]
ADD Al,BL
JNC L1
INC CL
L1:MOV [SI+2],AL
MOV [SI+3],CL
INT 03
```
Replace ADD with SUB
### Multiplication - 8 bit
```
MOV SI,1200H
MOV AL,[SI]
MOV BL,[SI+1]
MUL BL
MOV [SI+2],AL
MOV [SI+3],AH
INT 03
```
replace MUL with DIV

## <a href="https://github.com/ShafeeqAhamedS/EXPERIMENT--02-Interfacing-a-Digital-output-LED-to-LPC2148-ARM-7-Microcontroller-">Exp-02-Interfacing-a-Digital-output-LED</a>

```

#include <lpc214x.h>

void delay_ms(unsigned int count)
{
  unsigned int j=0,i=0;
  for(j=0;j<count;j++)
  {
    for(i=0;i<3000;i++);
  }
}

int main() 
{
    PINSEL0 = 0x00000000;  //Configure the P1 Pins for GPIO;
    IO0DIR = 0xffffffff; //Configure the P1 pins as OUTPUT;

  while(1)
    {
       IO0SET = 0xffffffff;     // Make all the Port pins as high  
         delay_ms(1000);

       IO0CLR = 0xffffffff;     // Make all the Port pins as low  
         delay_ms(1000);
    }
}
```
![image](https://user-images.githubusercontent.com/118756330/205136711-cdb7f626-1339-4694-a8d2-9b3478161eeb.png)
## <a href="https://github.com/NITHISHKUMAR-P/Experiment--03-Half-Subtractor-and-Full-subtractor.git" target="_blank">Exp--03-Interfacing-a-Digital-INPUT-push-button</a>
```
#include <LPC214x.h>   // define LPC2148 Header file
#define led (1<<2)     // led macro for pin 2 of port0
#define sw (1<<10)     // sw macro for pin 10 of port0
int main(void)
{
	unsigned int x;
	IO0DIR|=(~sw);   // configure P1.24 - P1.31 as input
	IO0DIR|=led;     // configure P1.16 - P1.23 as output
	while(1)
	{
		x = IOPIN0 & sw;   //save status of sw in variable x
		if(x==sw)          // if switch open
		{
			IOCLR0|=led; // LED off
		}
		else               // if switch close
		{
			IOSET0 = led;  // LED on
		}
	}
}
```
![image](https://user-images.githubusercontent.com/118756330/205137379-70b76b06-b2a7-465b-983d-3cba5eb3e43c.png)

## <a href="https://github.com/ShafeeqAhamedS/EXP-04-Interfacing-a-16X2-type-LCD-display-to-LPC2148-ARM-7-Microcontroller-">Exp-4-Interfacing a 16X2 type LCD display</a>

```
#include <lpc214x.h>
#include <stdint.h>
#include <stdlib.h>
#include <stdio.h>

void delay_ms(uint16_t j) /* Function for delay in milliseconds  */
{
    uint16_t x,i;
	for(i=0;i<j;i++)
	{
    for(x=0; x<6000; x++);    /* loop to generate 1 millisecond delay with Cclk = 60MHz */
	}
}

void LCD_CMD(char command)
{
	IO0PIN = ( (IO0PIN & 0xFFFF00FF) | (command<<8) );
	IO0SET = 0x00000040; /* EN = 1 */
	IO0CLR = 0x00000030; /* RS = 0, RW = 0 */
	delay_ms(2);
	IO0CLR = 0x00000040; /* EN = 0, RS and RW unchanged(i.e. RS = RW = 0) */
	delay_ms(5);
}

void LCD_INIT(void)
{
	IO0DIR = 0x0000FFF0; /* P0.8 to P0.15 LCD Data. P0.4,5,6 as RS RW and EN */
	delay_ms(20);
	LCD_CMD(0x38);  /* Initialize lcd */
	LCD_CMD(0x0C);   /* Display on cursor off */
	LCD_CMD(0x06);  /* Auto increment cursor */
	LCD_CMD(0x01);   /* Display clear */
	LCD_CMD(0x80);  /* First line first position */
}

void LCD_STRING (char* msg)
{
	uint8_t i=0;
	while(msg[i]!=0)
	{
		IO0PIN = ( (IO0PIN & 0xFFFF00FF) | (msg[i]<<8) );
		IO0SET = 0x00000050; /* RS = 1, , EN = 1 */
		IO0CLR = 0x00000020; /* RW = 0 */
		delay_ms(2);
		IO0CLR = 0x00000040; /* EN = 0, RS and RW unchanged(i.e. RS = 1, RW = 0) */
		delay_ms(5);
		i++;
	}
}

void LCD_CHAR (char msg)
{
		IO0PIN = ( (IO0PIN & 0xFFFF00FF) | (msg<<8) );
		IO0SET = 0x00000050; /* RS = 1, , EN = 1 */
		IO0CLR = 0x00000020; /* RW = 0 */
		delay_ms(2);
		IO0CLR = 0x00000040; /* EN = 0, RS and RW unchanged(i.e. RS = 1, RW = 0) */
		delay_ms(5);
}

int main(void)
{

	LCD_INIT();
	LCD_STRING("Sample Text");//first line
	LCD_CMD(0xC0);
	LCD_STRING("PM LAB");//second line

	return 0;
};
```
![image](https://user-images.githubusercontent.com/118756330/205137850-311f90e6-93a2-4397-9b23-76c73031fea6.png)

## <a href="https://github.com/shafeeqahameds/Experiment--05-4X4-keypad-interface-with-LPC2148">Exp 05-Interfacing a 4X4 keypad</a>
```
#include <lpc21xx.h> 
#define RS (1<<0)
#define RW (1<<1)
#define E (1<<2)

void LCD_command(unsigned char command);
void  delay_ms(unsigned char time);
void LCD_data(unsigned char data);
void LCD_init() ;


int main(void)
{
 //PINSEL1 = 0x00000000;  //Configure PORT0 as GPIO
 //PINSEL2 = 0X00000000;  //Configure PORT1 as GPIO
 IODIR1= 0x00780000; //Configure P1.18, P1.17, P1.16 as output(for rows and column)
 IODIR0= 0x00FF0007;  //Configure P0.23 - P0.16 as output for lcd data & P0.0,P0.1,P0.2 for lcd control lines.
 LCD_init();    //Initialize LCD 16x2
 LCD_command(0x01); 
 while(1)
   {
      IOCLR1|=(1<<19);               //Making row1 LOW
      IOSET1|=(1<<20)|(1<<21)|(1<<22); //Making rest of the rows '1'
      if(!(IOPIN1&(1<<16)))             //Scan for key press
       {
        while(!(IOPIN1&(1<<16)));
        LCD_data('1');                          
       }
      if(!(IOPIN1&(1<<17)))
       {
         while(!(IOPIN1&(1<<17)));
          LCD_data('2'); 
       }
      if(!(IOPIN1&(1<<18)))
       {
         while(!(IOPIN1&(1<<18)));
          LCD_data('3'); 
       }
      IOCLR1|=(1<<20);
      IOSET1|=(1<<21)|(1<<22)|(1<<19);
      if(!(IOPIN1&(1<<16)))
{
        while(!(IOPIN1&(1<<16)));
         LCD_data('4'); 
      }
      if(!(IOPIN1&(1<<17)))
{
        while(!(IOPIN1&(1<<17)));
         LCD_data('5'); 
     }
      if(!(IOPIN1&(1<<18)))
{
        while(!(IOPIN1&(1<<18)));
         LCD_data('6'); 
     }
      IOCLR1|=(1<<21);
      IOSET1|=(1<<22)|(1<<20)|(1<<19);
      if(!(IOPIN1&(1<<16)))
{
        while(!(IOPIN1&(1<<16)));
         LCD_data('7'); 
     }
      if(!(IOPIN1&(1<<17)))
{
       while(!(IOPIN1&(1<<17)));
        LCD_data('8'); 
    }
      if(!(IOPIN1&(1<<18)))
{
        while(!(IOPIN1&(1<<18)));
         LCD_data('9'); 
}
      IOCLR1|=(1<<22);
      IOSET1|=(1<<19)|(1<<20)|(1<<21);
      if(!(IOPIN1&(1<<16)))
{
        while(!(IOPIN1&(1<<16)));
         LCD_data('*'); 
}
      if(!(IOPIN1&(1<<17)))
{
        while(!(IOPIN1&(1<<17)));
         LCD_data('0'); 
}
      if(!(IOPIN1&(1<<18)))
{
        while(!(IOPIN1&(1<<18)));
         LCD_data('#'); 
} 
   }
}


//Function to generate software delay
//Calibrated to 1ms
void  delay_ms(unsigned char time)    
{  
 unsigned int  i, j;
 for (j=0; j<time; j++)
 {
  for(i=0; i<8002; i++)
  {
  }
}
}

void LCD_command(unsigned char command)
{
 IOCLR0 = 0xFF<<16; // Clear LCD Data lines
 IOCLR0=RS;     // RS=0 for command
 IOCLR0=RW;     // RW=0 for write
 IOSET0=command<<16; // put command on data line
 IOSET0=E;   // en=1 
 delay_ms(10) ;   // delay
 IOCLR0=E;    // en=0
}

void LCD_data(unsigned char data)
{
 IOCLR0 = 0xFF<<16; // Clear LCD Data lines
 IOSET0=RS;     // RS=1 for data
 IOCLR0=RW;     // RW=0 for write
 IOSET0= data<<16;  // put command on data line
 IOSET0=E;   //en=1 
 delay_ms(10) ;    //delay
 IOCLR0=E;   //en=0
 }

void LCD_init()
{
 LCD_command(0x38); //8bit mode and 5x8 dotes (function set)
 delay_ms(10) ;   // delay
 LCD_command(0x0c); //display on, cursor off, cursor char blinking off(display on/off)
 delay_ms(10) ;   // delay
    LCD_command(0x0e);  //cursor increment and display shift(entry mode set)
    delay_ms(10) ;   // delay
 LCD_command(0x01);  //clear lcd (clear command)
 delay_ms(10) ;   // delay
 LCD_command(0x80); 
  delay_ms(10) ;//set cursor to 0th location 1st lne
 
}
```
![image](https://user-images.githubusercontent.com/118756330/205138630-97d4e18f-2ed4-42c6-88f0-14b8b3e03c50.png)

## <a href="https://github.com/ShafeeqAhamedS/Exp-06-Configuration-of-ADC-for-converting-analog-to-digital-signals" target="_blank">Exp-6-converting analog to digital signals</a>
```
#include <lpc214x.h>
#include "LCD.h"
#include "ADC.h"
unsigned int val;
/*void delay_ms(unsigned int count)
{
	unsigned int i=0,j=0;
	for(j=0;j<count;j++)
	{
		for(i=0;i<count;i++);
	}
}*/
int main()
{
	IO1DIR = 0xffffffff;
	IO0DIR = 0x00000000;
	PINSEL0 = 0x0300;
	VPBDIV = 0x02;
	lcd_init();
	show(" ADC Value:");
	while(1)
	{
		cmd(0x8b);
		//delay_ms(1000);
		val=adc(0,6);
		dat((val/1000)+48);
		dat(((val/100)%10)+48);
		dat(((val/10)%10)+48);
		dat((val%10)+48);
	}
}
```
![image](https://user-images.githubusercontent.com/118756330/205138941-b78b4c95-ef6e-4b80-af95-a59dc8c3d154.png)

## <a href="https://github.com/ShafeeqAhamedS/Ex.-No.-7-Interfacing-LM35-Temperature-sensor-and-calculate-the-sensitivity-of-the-output">Ex 07-Interfacing LM35 Temperature sensor</a>
```
#include <lpc214x.h>
#include "LCD.h"
#include "ADC.h"
unsigned int val;
/*void delay_ms(unsigned int count)
{
	unsigned int i=0,j=0;
	for(j=0;j<count;j++)
	{
		for(i=0;i<count;i++);
	}
}*/
int main()
{
	IO1DIR = 0xffffffff;
	IO0DIR = 0x00000000;
	PINSEL0 = 0x0300;
	VPBDIV = 0x02;
	lcd_init();
	show(" ADC Value:");
	while(1)
	{
		cmd(0x8b);
		//delay_ms(1000);
		val=adc(0,6);
		dat((val/1000)+48);
		dat(((val/100)%10)+48);
		dat(((val/10)%10)+48);
		dat((val%10)+48);
	}
}
```
### LCD.h
```
//lcd.h
#define bit(x) (1<<x)

void lcd_init(void);
void cmd(unsigned char a);
void dat(unsigned char b);
void show(unsigned char *s);
void lcd_delay(void);

void lcd_init()
{
	cmd(0x38);
	cmd(0x0e);
	cmd(0x01);
	cmd(0x06);
	cmd(0x0c);
	cmd(0x80);
}

void cmd(unsigned char a)
{
	IO1CLR=0xFF070000;
	IO1SET=(a<<24);
	IO1CLR=bit(16);				//rs=0
	IO1CLR=bit(17);				//rw=0
	IO1SET=bit(18);			  	//en=1
	lcd_delay();
	IO1CLR=bit(18);			   	//en=0
}

void dat(unsigned char b)
{
	IO1CLR=0xFF070000;
	IO1SET=(b<<24);
	IO1SET=bit(16);				//rs=1
	IO1CLR=bit(17);				//rw=0
	IO1SET=bit(18);			   	//en=1
	lcd_delay();
	IO1CLR=bit(18);			   	//en=0
}

void show(unsigned char *s)
{
	while(*s) {
		dat(*s++);
	}
}

void lcd_delay()
{
	unsigned int i;
	for(i=0;i<=3000;i++);
}
```
### ADC.h
```
//adc.h
unsigned int val;
unsigned int adc(int,int);



unsigned int adc(int no,int ch)
{
	switch(no)									  //select adc
	{
		case 0:	AD0CR=0x00200600|(1<<ch);		  //select channel
				AD0CR|=(1<<24);				  //start conversion
				while((AD0GDR& (1<<31))==0);
				val=AD0GDR;
				break;

		case 1:	AD1CR=0x00200600|(1<<ch);		  //select channel
				AD1CR|=(1<<24);				  //start conversion
				while((AD1GDR&(1<<31))==0);
				val=AD1GDR;
				break;
	}
	val=(val >> 6) & 0x03FF; 					 // bit 6:15 is 10 bit AD value

	return val;
}
```
![image](https://user-images.githubusercontent.com/118756330/205139339-97106071-80e1-43d9-97d0-e90cb41d162b.png)
## <a href="https://github.com/ShafeeqAhamedS/Interfacing-Seven-segment-display-with-lpc2148">Exp 08-Interfacing seven segment display</a>
```
#include <LPC214x.h>
unsigned char dig[]={0x88,0xeb,0x4c,0x49,0x2b,0x19,0x18,0xcb,0x8,0x9,0xa,0x38,0x9c,0x68};
	void delay(unsigned int count)
	{
		int j=0,i=0;
		for(j=0;j<count;j++)
		{
			for(i=0;i<120;i++);
		}
	}
	int main(void)
	{
		unsigned char count=0;
		unsigned int i=0;
		IO0DIR|=(1<<11);//Set Digit control lines as Outputs
		IO0SET=(1<<11);
		IO0DIR|=0x007F8000;
		while(1)
		{
			count++;
			if(count==16)count=0;
			for(i=0;i<800;i++)//change to inc/dec speed of count
			{
				IO0CLR=0x007F8000;
				IO0SET=(dig[count]<<15);
				delay(200);
			}
		}
	}
```
![image](https://user-images.githubusercontent.com/118756330/205139636-4c05cb17-4211-4fad-bb0d-88bcb258b992.png)
## <a href="https://github.com/shafeeqahameds/Experiment--09-Configuring-UART-in-LPC2148-for-serial-data-transmission-">Exp 09-Configuring UART for seral data transmission</a>
```
#include <LPC213x.H> // LPC21xx definitions */
char a;
void uart0_init(){
PINSEL0 = 0x00000005; // Enable RxD0 and TxD0 */
U0LCR = 0x83; // 8 bits, no Parity, 1 Stop bit */
U0DLL = 97; // 9600 Baud Rate @ 15MHz VPB Clock */
U0LCR = 0x03; // DLAB = 0 */
}
void uart0_putc(char c){
while(!(U0LSR & 0x20)); 
	U0THR = c; // Send character
}
int uart0_getc (void) {
while (!(U0LSR & 0x01));
return (U0RBR);
}
int main (void) {
uart0_init();
while (1) {
a=uart0_getc();
uart0_putc(a);
}
}
```
![image](https://user-images.githubusercontent.com/118756330/205139923-ac5c0db4-b232-4d00-9cac-be0ac8dbeaff.png)
![image](https://user-images.githubusercontent.com/118756330/205139947-0405a0b7-e449-47d6-96ef-9710ae995a78.png)
