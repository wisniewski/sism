/*
* Mariusz Wisniewski 254019
* 01-03-2014, Systemy i sterowniki mikroprocesorowe, labolatorium 02
* Opis programu: Zabezpieczenie drzwi 4-cyfrowym kodem (2589 i enter)
* Program ma nastepujace cechy:
* - po 3 nieudanych probach wpisania kodu blokada na 1 minute (sygnalizowane naprzemiennym miganiem LEDow co 1s)
* - pierwsze 4 cyfry kodu to 0-9 (blokada innych przyciskow), zatwierdzenie wylacznie Enterem (blokada innych przyciskow)
* - sygnalizowaniy numer wpisanej liczy (1 libcza, 2 liczby, 3 liczby, 4 liczby i oczekiwanie na Enter)
* - program oparty na skanowaniu klawiatury 4x3 (moj algorytm)
* - zaimplementowany debouncing oraz wpisanie nastepnej wartosci DOPIERO PO zwolnieniu przycisku
* - schemat wykonany w programie Eagle, dostepny w zalaczniku
*
* Software: Atmel Studio 6.1
* Hardware: ATmega32, zestaw: ZL15AVR
*/

#include <avr/io.h>
#include <avr/interrupt.h>

const uint8_t code_ok[5] = {2, 6, 10, 11, 15}; //correct code
uint8_t keyboard_scan(void); //function declaration
void keyboard_scan_code(uint8_t *, uint8_t *, uint8_t []);

ISR(TIMER0_COMP_vect)
{
	static uint16_t time, time_door, time_attempt;
	static uint8_t code[5], i, status, turn_on, counter, attempt, minute, oldkey;
	uint8_t j, key;

	if(!time) //after 1 ms
	{
		if(attempt==3) //3 bad attempts
		{
			if(!time_attempt)
			{
				time_attempt=1000; //1 second
				minute++;
				PORTA ^= 0xff; //flashing LEDs
				if(minute==10) //after 60 seconds
				{
					attempt=0; //clear all
					minute=0;
				}
			}
			else
			time_attempt--;
		}
		else
		{
			if(!counter) //debouncing
			{
				key = keyboard_scan();
				counter = 100;
				PORTA = (1 << i); //show actual number
				if((key != 255) && (key != oldkey)) //if we are pressing button - nothing will happen
				{
					if((i!=4) && (keyboard_scan() != 15)) //enter only numbers from 0-9
					keyboard_scan_code(&i,&status,code); //enter 4-digit code
					else if((i==4) && (keyboard_scan() == 15)) //last key must be enter!
					keyboard_scan_code(&i,&status,code);
				}
				oldkey=keyboard_scan(); //next number only after release button !!!
				
			}
			else if(counter)
			counter--;
			
			if(status == 5) //if code is correct!
			turn_on = 1; //turn on doors

			if(turn_on)
			{
				PORTA = 255; //open doors
				time_door++;
				if(time_door == 5000) //after 5 sec close doors
				{
					time_door = 0;
					turn_on = 0;
					PORTA = 0;
					attempt=0;
				}
			}
			
			if(i>4) //prepare for next code
			{
				i = 0;
				status = 0;
				for(j=0; j<5; j++) //clear current code
				code[j]=0;
				attempt++;
			}
		}
		time = 10; //1 ms
	}
	else
	time--;
}

int main(void)
{
	DDRA = 0xff;
	DDRC = 0x0f; //LSB output
	PORTC = 0xf0; //MSB input

	TCCR0 |= (1<<WGM01) | (1<<CS01); //ctc mode, prescaler: 8
	TIMSK |= (1<<OCIE0); //compare output mode
	OCR0 = 99; //10 kHz = 100 us
	sei();

	while(1);

	return 0;
}

uint8_t keyboard_scan(void) //read from 4x3 matrix keyboard
{
	uint8_t row, col;

	for(row = 0; row < 4; row++)
	{
		PORTC = (PORTC | 0x0f) ^ (1<<row); //choose row
		asm volatile ("nop");
		for(col = 0; col < 3; col++) //only 3 cols
		{
			asm volatile ("nop");
			if(!(PINC & (0x10 << col)))
			return 1 + (4*row + col); //return number 1-3, 5-7, 9-11 and 13-15
		}
	}
	return 255; //if nothing pressed
}

void keyboard_scan_code(uint8_t *j, uint8_t *stat, uint8_t cod[])
{
	cod[*j] = keyboard_scan(); //enter digit to array
	if(cod[*j] == code_ok[*j])
	*stat=*stat+1; //increase status if number is ok
	*j=*j+1; //increase number of digit
}