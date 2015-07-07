---
layout: post
Title: Arduino controlled EPSON TM-T88P Printer
Date: 2010-06-20 17:00  
tags: [Electronics, Open Source, Programing]
---

The EPSON TM-T88P is a parallel controlled printer. It came to be in my
possession after going along to a kind of computer jumble sale 5 quid got me this:

![Epson printer at #bcne3](http://dl.dropbox.com/u/78443198/apps/scriptogram/epson.jpg)

Centronics Handsake
-------------------

It turns out there are a number of ways to communicate with printers and
the Epson supports a number of these methods. I decided to go with the
Centronics standard AKA Compatibility Mode since its about as basic as
you can get. Heres a diagram from the excellent
[http://www.beyondlogic.org][]

![parallel port handshake](http://dl.dropbox.com/u/78443198/apps/scriptogram/parallelport_img_0.jpg)

Centronics Handshake Diagram from beyondlogic.org

Think of the handshake as; you wait for the printer to be free (busy
low) enter the number you want to send (set the 8 bits of your byte on
the data lines) then pull the send lever old cash register style
(strobe low wait stobe high). Its not a partuclay dificult concept but I
found the diagram hard to understand becuase for some reason I thought
it had to be more complicated than it actualy was.

Wiring Up
---------

Nothing too complex here the table below show the Socket to Arduino
linkup:

<table width="100%">
<tbody>
<tr>
<th colspan="2">
Socket

</th>
<th colspan="2">
Arduino

</th>
</tr>
<tr>
<th>
Pin

</th>
<th>
Name

</th>
<th>
Pin

</th>
<th>
pinMode

</th>
</tr>
<tr>
<td>
1

</td>
<td>
Strobe

</td>
<td>
13

</td>
<td>
OUTPUT

</td>
</tr>
<tr>
<td>
2

</td>
<td>
D0

</td>
<td>
2

</td>
<td>
OUTPUT

</td>
</tr>
<tr>
<td>
3

</td>
<td>
D1

</td>
<td>
3

</td>
<td>
OUTPUT

</td>
</tr>
<tr>
<td>
4

</td>
<td>
D2

</td>
<td>
4

</td>
<td>
OUTPUT

</td>
</tr>
<tr>
<td>
5

</td>
<td>
D3

</td>
<td>
5

</td>
<td>
OUTPUT

</td>
</tr>
<tr>
<td>
6

</td>
<td>
D4

</td>
<td>
6

</td>
<td>
OUTPUT

</td>
</tr>
<tr>
<td>
7

</td>
<td>
D5

</td>
<td>
7

</td>
<td>
OUTPUT

</td>
</tr>
<tr>
<td>
8

</td>
<td>
D6

</td>
<td>
8

</td>
<td>
OUTPUT

</td>
</tr>
<tr>
<td>
9

</td>
<td>
D7

</td>
<td>
9

</td>
<td>
OUTPUT

</td>
</tr>
<tr>
<td>
10

</td>
<td>
ACK

</td>
<td>
10

</td>
<td>
INPUT

</td>
</tr>
<tr>
<td>
11

</td>
<td>
BUSY

</td>
<td>
11

</td>
<td>
INPUT

</td>
</tr>
<tr>
<td>
25

</td>
<td>
GND

</td>
<td>
GND

</td>
<td>
n/a

</td>
</tr>
</tbody>
</table>
You only need to wire up one of the ground lines since they will all be
connected to ground in the printer anyway (lazy method) ! :)

Programming
-----------

Its worth pointing out at this stage that you might want to make up some
kind of parallel port debuger so you can get an idea of whats actualy
happening at a low level and that your program is producing the expected
output.

First lets setup the pins as in the table above we will also setup a
serial connection so we can do debugging.
{% highlight c %}
int strobe = 12;
int busy = 11;
int ack = 10;

void setup()   {
  pinMode(strobe, OUTPUT);
  //setup stobe high
  digitalWrite(strobe, HIGH);
  //setup handshake listeners
  pinMode(busy, INPUT);
  pinMode(ack, INPUT);
  for(int i=2;i<10;i++){
     pinMode(i, OUTPUT);
     //setup inital condition
     digitalWrite(i, LOW);
  }
  Serial.begin(115200);
  Serial.println("READY:");
}
{% endhighlight %}
To start with we can actual simplify the handsake further by ignoring
the ACK since BUSY should always go low when we are OK to send more
data. With this in mind we end up with 3 main steps:

1.  Wait for not busy
2.  Set Data
3.  Strobe

For the wait not busy function i decided to go with a simple poll, every
100 iterations it prints a ‘.’ so we can see in the terminal if its got
stuck.
{% highlight c %}
void pollWait(int port){
  int i = 0;
  while(digitalRead(port) == 1){
    i++;
    if(i>100){
      i=0;
     Serial.print(".");
    }
  }
}
{% endhighlight %}
Strobe function is nice and simple. Suposidly you can have any value
greater than 0.5 microseconds for the strobe and it will work. To keep
it simple i just went with 1ms.
{% highlight c %}
void doStrobe(){
    digitalWrite(strobe, LOW);
    delay(1);
    digitalWrite(strobe, HIGH);
}
{% endhighlight %}
Setting the datalines we use i+2 as the first bit is written to pin 2
etc.
{% highlight c %}
	void setLines(byte out){
	  //loop over data lines
	  for(int i=0;i<8;i++){
	    //check and set
	   if(bitRead(out, i) == 1){
	     //write to pint we start at 2 so +2
	     digitalWrite(i+2, HIGH);
	   }else{
	     //bit is 0 so set low
	     digitalWrite(i+2, LOW);
	   }

	  }
	}
	{% endhighlight %}
Now we use all these functions to send a byte to the printer
{% highlight c %}
void dataHandShake(byte a){
    pollWait(busy);
    setLines(a);
    doStrobe();
}
{% endhighlight %}
Finaly we can make a small program to print somthing. Its worth making
sure things only happen once… you dont want your printer spewing paper
while you try and turn it off!

Add the following global:
{% highlight c %}
boolean go = true;
{% endhighlight %}
Then quick and dirty hello world:

{% highlight c %}
void loop()
{
 //so we dont end up with a constant stream
 if(go){
     dataHandShake('H');
     dataHandShake('e');
     dataHandShake('l');
     dataHandShake('l');
     dataHandShake('o');
     dataHandShake(' ');
     dataHandShake('W');
     dataHandShake('o');
     dataHandShake('r');
     dataHandShake('l');
     dataHandShake('d');
     //send Line feed so it prints the line
     dataHandShake(0x0A);
     go = false;
     Serial.println("DONE");
   }
}
{% endhighlight %}

We now end up with something that looks like this:

![Hello World!](http://dl.dropbox.com/u/78443198/apps/scriptogram/hello-world.jpg)

Thats all for now in a future post (hopefully!) I will be looking at
more advanced printing.

  [http://www.beyondlogic.org]: http://www.beyondlogic.org
