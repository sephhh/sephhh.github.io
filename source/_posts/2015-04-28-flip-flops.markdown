---
layout: post
title: ".. flip-flops .."
date: 2015-04-28 16:50:22 -0400
comments: true
categories: [Ruby]
---

Flip-flops are a funky and fairly obscure feature in Ruby. In fact, there have been <a href="https://bugs.ruby-lang.org/issues/5400">murmers</a> about flip-flops being removed in future versions of Ruby altogether. Most folks probably wouldn't notice if these things disappeared from Ruby tomorrow, but this quirky mechanism can be nifty and fun to play around with once you get the hang of it.

I first came across flip-flops in Konstantin Haase's <a href="https://vimeo.com/61087285">talk at Ruby Australia</a> where he talks about obfuscation and presents his 6-line implementation of Sinatra. In the talk, Haase presents the following code and asks what it will produce.

```ruby
1.upto 10 do |i|
  puts i if i == 3 .. i == 5
end
```

Can you guess the output yourself? Any idea why? The key is the flip-flop, which is used here in the if condition. Let's take a look. 


<h3>Making Flippy Floppy</h3>

In electronics, a flip-flop is a circuit that can have one of two states, On or Off (1 or 0). Along these lines, Ruby flip-flops act like a switch that can be flipped on and off (or like a gate that can be either open or closed). The syntax for a flip-flop uses a range operator `(..)` but the range spans between two conditions instead of two numbers. If you can hold those two ideas in your head, ranges and switches, you can sort of see how the flip-flop behaves like a switch that turns On for a certain range before turning Off again. What determines whether the if clause gets evaluated is not the truth of either condition, but the overall state of the flip-flop: On or Off. Here's the flip-flop from earlier again.

```ruby
1.upto 10 do |i|
  puts i if i == 3 .. i == 5
end
```

When the first condition is met (`i==3`), you can imagine the switch flipping On, so the conditional if clause is executed, and the code prints `i`. That's all well and good--just what you'd expect from an if statement when the condition is true. But here's the kicker: that if clause is executed the next time through the loop too, when `i` is 4. The flip-flop has been switched On, and that On or `true` state persists through subsequent interations. The if clause (in this case, `puts i`) continues to run on each iteration until the <em>second</em> condition is true (in this case, `i == 5`). That switches the flip-flop back off. If on a later iteration the first condition evalutes to true again, the flip-flop switches back On, and stays On, executing the if clause until the second condition is true again, and so on indefinitely.

So now maybe it makes sense (kind of) how the code actually evaluates:

```ruby
1.upto 10 do |i|
  puts i if i == 3 .. i == 5
end

# 3
# 4
# 5
```

In case you were starting to feel less confused, flip-flops can have either two or three dots between the two conditions, and they behave a bit differently in each case. If you use two dots, the second condition is evaluated <em>immediately</em> as soon as the first condition evaluates to `true`. That means that if both conditions are true on the same iteration, the switch will turn on and then immediately off again, executing the conditional clause exactly once:

```ruby
1.upto 10 do |i|
  puts i if i == 3 .. i == 3
end

# 3
```

With three dots, on the other hand, the second condition won't be evaluated until the iteration <em>after</em> the first condition becomes true. 

```ruby
1.upto 10 do |i|
  puts i if i == 3 ... i == 3
end

# 3
# 4
# 5
# ... all the way on up to 10
```

So in this code, instead of starting and then stopping at three, the flip-flop just switches On and never switches Off. It keeps printing `i` for the rest of the iterations (again, these examples come from Haase's talk)

<h3>Flizz Fluzz</h3>

If you feel like getting deeper into the weeds, here's a devious flip-flopping version of fizzbuzz written by <a href="https://gist.github.com/judofyr/3230984">Magnus Holm</a>:

```ruby
a=b=c=(1..100).each do |num|
  print num, ?\r,
    ("Fizz" unless (a = !a) .. (a = !a)),
    ("Buzz" unless (b = !b) ... !((c = !c) .. (c = !c))),
    ?\n
end
```

There's a very nice detailed breakdown of this code on <a href="https://juliansimioni.com/blog/deconstructing-fizz-buzz-with-flip-flops-in-ruby/">Julian Simioni's blog</a>, but here are a few highlights:
<ul>
    <li>Notice how the flip-flop conditions actually include assignments--the truth value of the variable <code>a</code> is reversed each time one of those conditions is evaluated.</li>
    <li>Which condition is evaluated depends on the state of the flip-flop--the first condition will only be evaluated if the flip-flop is Off, and the second condition will only be evaluated if it's On. This very craftily makes it so the flip-flop is Off every third iteration (which makes it print "Fizz").</li>
    <li>The solution also very sneakily uses <code>\r</code>, which is a carriage return. That essentially moves the cursor back to the very front of the line without going to a new line, which means when you print you'll just overwrite any characters written on that line already. (This way the solution doesn't have to worry about skipping any numbers: it just overwrites them)</li>
</ul>

<h3>Trying it out</h3>

I decided to give flip-flops a try on my own with a simple function meant to highlight or otherwise mess with certain marked portions of text. The idea is, whenever a line starts with a certain marker (like `~`), my function will do something to change or highlight each line of text, until that highlighting is turned off by a subsequent marker. Here it goes:

```ruby

def highlight_file(filepath, marker)
  #read in the file
  orig_lines = File.readlines(filepath)
   updated_lines = orig_lines.collect do |line|

    #Turn the flip-flop on or off whenever you hit a marker
    if (line.start_with?(marker))...line.start_with?(marker)
      #as long as the flip-flop is on, it yields each line to a block to mess with it
      if block_given?
        yield(line)
      else
        #or upcases text if no block is given
        line.upcase
      end
    else
      #if the flip-flop is off, leave the line untouched
      line
    end
  end
  #join the lines and delete the marker from the file
  final_text = updated_lines.join.gsub(marker, "")
  #write the result to a file
  File.open("new_highlighted_file.txt", 'w') {|f| f.write(final_text) }
end
```

The function reads in a file and takes a string marker as an argument. Whenever a line starts with the marker, the flip-flop turns on, and the function starts yielding each line to a block. The block can do whatever with the line to modify and replace it. If no block is given, the function just upcases the line. When the function reaches another line that starts with a marker, the flip-flop switches back off and just returns lines untouched until it switches back on again.

Here's some sample text, which uses '~' as a marker:

```text
Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod
tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam.</br>

~
Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod
tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam,
quis nostrud.</br>
~
Lorem ipsum</br>

~Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod
tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam,
~proident, sunt in culpa qui officia deserunt mollit anim id est laborum.</br> 

Tadah!
```

And here's a sample run:

```ruby
highlight_file("./test_file_1.txt", "~")
```

And here's the output:

```text
Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod
tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam.</br>


LOREM IPSUM DOLOR SIT AMET, CONSECTETUR ADIPISICING ELIT, SED DO EIUSMOD
TEMPOR INCIDIDUNT UT LABORE ET DOLORE MAGNA ALIQUA. UT ENIM AD MINIM VENIAM,
QUIS NOSTRUD.</BR>

Lorem ipsum</br>

LOREM IPSUM DOLOR SIT AMET, CONSECTETUR ADIPISICING ELIT, SED DO EIUSMOD
TEMPOR INCIDIDUNT UT LABORE ET DOLORE MAGNA ALIQUA. UT ENIM AD MINIM VENIAM,
PROIDENT, SUNT IN CULPA QUI OFFICIA DESERUNT MOLLIT ANIM ID EST LABORUM.</BR> 

Tadah!
```

I didn't pass the method a block, so it just upcased the text surrounded by markers. Here's another sample run, this time with a block that adds `<strong>` and `<em>` tags to the lines between the markers:

```ruby
highlight_file("./test_file_1.txt", "~") {|line|
 "<strong><em>#{line}</em></strong>"}
 ```
 And here's the HTML output:

<blockquote>
 Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod
tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam.</br>

<strong><em>
</em></strong><strong><em>Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod
</em></strong><strong><em>tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam,
</em></strong><strong><em>quis nostrud.</br>
</em></strong><strong><em>
</em></strong>Lorem ipsum</br>

<strong><em>Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod
</em></strong><strong><em>tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam,
</em></strong><strong><em>proident, sunt in culpa qui officia deserunt mollit anim id est laborum.</br> 
</em></strong>
Tadah!
</blockquote>

<h3>Conclusion</h3>
My function could probably use some cleaning up, but I think the flip-flop itself actually makes decent sense here. It's sees each marker as a switch, which I use to turn on or off certain behavior. It's not a tool I'll likely be using too often but the logic of it is fun to explore, and you never know when a useful application might turn up.


<div id="container">
  <div id="background-text">
    F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . F L I P - F L O P S . . 
  </div>
  <div>
    <div class="big-caption">
      FIRST I WAS LIKE
    </div>
    <iframe src="//giphy.com/embed/dk3KF9iSGU9nG" width="240" height="247" frameBorder="0" style="max-width: 100%" class="giphy-embed" webkitAllowFullScreen mozallowfullscreen allowFullScreen></iframe>
    <div class="big-caption">
      BUT NOW I'M LIKE
    </div>
    <iframe src="//giphy.com/embed/cWV5NrLzjpke4" width="240" height="350" frameBorder="0" style="max-width: 100%" class="giphy-embed" webkitAllowFullScreen mozallowfullscreen allowFullScreen></iframe>
  </div>
</div>
