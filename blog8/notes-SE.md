# GUI in Terminal

In this article, I will show how to create a GUI in a terminal. I am going to use ruby for this article because it is very easy to understand.
So, lets start.

# Draw a pixel in Terminal

As such, termianl is character cell i.e. it receives keystrokes and then outputs it to terminal. So, there is nothing like pixel access in terminal. 

So, we will declare our own defination of the pixel.

To define the background color for a character in terminal we use the following escape sequence.


`\e[<color>m`

everything after this sequence has a background color as specified. Following colors are supported:

```bash
	49 - Default background color (usually black or blue)
	40 - Black
	41 - Red
	42 - Green
	43 - Yellow
	44 - Blue
	45 - Magenta/Purple
	46 - Cyan
	47 - Light Gray 
	100 - Dark Gray 
	101 - Light Red
	102 - Light Green 
	103 - Light Yellow
	104 - Light Blue 
	105 - Light Magenta/Pink
	106 - Light Cyan 
	107 - White
```

e.g. the following statement will display text in white background.

```bash
$ printf "\e[107mThis is a text in white"

```

But this also considers any statement after it that I don't want to be in the white. So, to terminate any formatting we use the following escape sequence


`\e[0m`

Since, `space` is also a character why not use it. Space will occupy one character position without showing any thing. So, this will make our defination of the pixel.

Thus, defination of our pixel will be

`\e[<color>m \e[0m`

To move to any cell on terminal following sequence is used

`\e[<x>;<y>H`

Let's code it

```ruby
	def drawPixel(x, y, color)
		printf("\e[#{x};#{y}H\e[#{color}m \e[0m")
	end
```

Now, let's try to draw a simple rectangle with this defination of the pixel

```ruby
def drawStroke(x, y, length, color: "107", ver_hor: 0)
	if ver_hor.eql?0
		for i in (0..length-1)
			drawPixel(x, y + i, color)
		end
	elsif ver_hor.eql?1
		for i in (0..length-1)
			drawPixel(x+i, y, color)
		end
	end
end

printf("\e[2J")
drawStroke(1, 1, 10)
drawStroke(1, 1, 4, ver_hor: 1)
drawStroke(4, 1, 10, ver_hor: 0)
drawStroke(1, 10, 4, ver_hor: 1)
	
```
Lets have a quick go through

`drawStroke` function will be a helper function that will help to draw vertical and horizontal strokes depending upon the variable `ver_hor`

`\e[2J` will be used to clear the screen before displaying anything.

Rest of the lines are used to display a rectangle of size `10 x 4`

So, now you know how to draw a simple rectangle in terminal, with a little modification you can draw any shape and picture e.g I have drawn this
<img src="https://github.com/convosyn/gui-term/raw/master/examples/sample.gif">
Find the code for this [here](https://github.com/convosyn/gui-term/blob/master/examples/flappyAnime.rb)