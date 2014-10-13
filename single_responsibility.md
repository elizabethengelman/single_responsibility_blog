Single Responsibility Principle
===============================
The SRP is first of Uncle Bob's SOLID principles, and is perhaps one of
the most important principles [DIFFERENT WORD FOR PRINCIPLES] to follow when writing good, clean
software. By adhering to the SOLID principles, we can set ourselves up
for success, since we're creating code that is more easily maintaned,
and extended as our project evolves. In a nutshell, the SRP states that
each software entity we create should have one, and only one reason for changing.

Traditionally, we think of a 'responsibility' as a state of dealing with
or having control over a specific issue. So, we may think of a software entity's responsibility,
as needing to take care of a very specific problem within our system. Though this definition is true in software design,
we can also introduce the idea that a responsibility is a *reason for change*.

In other words, our software entities (classes, methods, etc) should
never need to change for a reason that is not related to its central
purpose.

In order to have a better understanding of this principle, let's consider a Tic Tac Toe program, written in Ruby. This program has a ```Board```
class:

```
class Board

  def generate_blank_board
    #generates an empty board state
  end

  def display_current_board_in_terminal
    #displays the board to the terminal
  end

  def update_board
    #updated the board's state
  end
end
```

Our ```Board``` has methods that generate, display and update a Tic Tac Toe board. At first glance, this class may
appear to adhere to the Single Responsibility Principle without a problem; each of the three methods is
specifically related to a Board object. However, let's look a little closer, and start to view this class in
terms of *reasons for change*. We can see that both the ```generate_blank_board``` and
```update_board``` methods are dealing specifically with the **state** of
the board itself. While, ```display_current_board``` deals with
**displaying** the board to the user. At the moment we have two
responsibilities, or two *reasons for change*: a board's state (if a
space is empty, or if it is filled with an X or O), and a board's user
interface.

Leaving these two separate responsibilities in the ```Board``` class,
would essentially be coupling the idea of a ```Board```'s state with
it's display. If we ever want to change how a board's **state is
stored** or how a board is **displayed**, we would be dealing with two
very different responsibilities.  Say our requirements change, and now
instead of displaying our game on the command line, we also want to
display the it in the browser as well. While creating a browser UI,
there would be a great chance that we could introduce a bug into the
"state keeping" section of our module. Furthermore, since these two
responsibilities are coupled into one module, there is also a chance
that this one change could break another part of our game system. In
essence, this is a fragility.

In Practical Object-Oriented Design in Ruby by Sandi Metz, she describes
one of my favorite tips for writing software: when describing what a class (or any
other software entity) does in one sentence, if you use the words "and" or "or", you're most
likely breaking the Single Responsibility Principle. If you use the
word "and", your class has at least two responsibiliites. If you use the
word "or", the class likely has more than one responsibility, and there's
a good chance the two responsibilities aren't related to one another.
Let's try this out on our ```Board``` class: It keeps track of a Tic Tac
Toe board's state **and** displays this state to the user. Busted!

We've now determined that our ```Board``` class does in fact
violate the Single Responsibility Principle, but why do we care?

In order to answer this question, let's dig back into our Tic Tac Toe
example. As our Tic Tac Toe game evolves, let's say that we
decide to extend our program to allow the user to play in a browser, in addition to our initial display in the terminal.
So, how could we accomplish this? Our first option could be to use the
same ```Board``` class that we defined above, and to add the new
functionality to it:

```
class Board

  def generate_blank_board
    #generates an empty board state
  end

  def display_current_board_in_terminal
    #displays the board on the command line
  end

  def display_current_board_in_browser
    #displays the current board in a brower
  end

  def update_board
    #updated the board's state
  end
end
```

Now we have a ```Board``` class that does exactly what we need it to... or does it?
If we look at this code a little deeper, we'll realize that the class
that may be responsible for rendering the game in the browswer, shouldn't have any need to
know anything about how the game is displayed in the terminal. But, the
way that we've currently written ```Board```, the browser displaying
logic is very tightly coupled with the terminal display logic:

class BrowserGame
  def play_the_game
    board = Board.new
    board.display_current_board_in_browser

    #other game logic
  end
end

Everytime we need to make a change to
```display_current_board_in_terminal``` for example, we will
be touching the ```Board``` class and thereby risk introducing a bug or some
sort of unintended side effect. This side effect could then potentially trickle
down [DIFFERENT word for trickle down] and also effect how the
```Board``` class behaves in the browser, even though it doesn't ever use
```display_current_board_in_terminal```!

This is perhaps one of the biggest flaws with reusing a class that
breaks the Single Responsibility Principle.

Our other option to reuse ```Board``` in its current state
would be to duplicate it, and make the small change that we need for the
terminal game (a new display method specifically for the browswer):

```
class BrowserBoard

  def generate_blank_board
    #generates an empty board state
  end

  def display_current_board_in_browser
    #displays the board to the browser
  end

  ef update_board
    #updated the board's state
  end
end
```

This choice is flawed as well. If we ever need to change the way a
```Board``` is updated, we would need to change the
```update_board``` method in both ```BrowserBoard``` and
```TerminalBoard```, which opens us up for double the chance to introduce a
bug or inconsistency.




So, how do we fix the ```Board``` module above? Instead of including
code to both **display** a board and **keep it's state**, let's split
those two pieces into two separate modules:
```
class BoardState
  def generate_blank_board
    #generate a board state
  end

  def update_board
    #updated the board's state
  end
end
```
```
class BoardPresenter
  def display_current_board
    #displays the board to the ui
  end
end
```

It's a simple change, and now we have two separate modules that can take
care of very different aspects of a Board. In the scenario about, where
an application may need to only display a board, we can use the
```BoardPresenter```. And when we need to keep a board's state, clearly
we can use ```BoardState```. Most likely, these two modules will be used
in conjunction with each other - when a board's state is updated using
```BoardState.update_board```, we could potentially pass it's result
into ```BoardPresenter.display_current_board```. This also allows us to
create more than one ```BoardPresenter``` object which we could then use
for different display requirements: command line, browser, etc.

Sources:
Practical Object-Oriented Design in Ruby by Sandi Metz
Agile Software Development, Principles, Patterns, and Practices by
Robert "Uncle Bob" Martin

IDEAS:
-a class should do the smallest possible thing
-describe what the class is doing in one sentence -> 'and' class likely
has more than one responsibility, 'or' -> the class has more than one
responsibility and they likely aren't even related

A benefit of creating code that is reusable is that we're able to reduce
our code duplication, and as we know, crafting software that is DRY is
yet another important [DIFFERENT WORD] tenant to follow when creating
software. By creating DRY code, we reduce the need to update FINISH this
sentence

