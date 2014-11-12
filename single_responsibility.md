Single Responsibility Principle
===============================

The Single Responsibility Principle (SRP) is the first of Uncle Bob's SOLID principles, and is perhaps the most important to follow when writing good, clean software. By adhering to the SOLID principles, we can set ourselves up for success in the future, since we're creating code that is more easily maintained, and extended as our project evolves. In a nutshell, the SRP states that each software entity we create should have one, and only one, reason for changing.

Traditionally, we think of a 'responsibility' as a state of dealing with or having control over a specific issue. So, we may think of a software entity's responsibility as needing to take care of a very specific problem within our system.  Though this definition is true in software design, we can also introduce the idea that a responsibility is a *reason for change*. In other words, our software entities (classes, methods, etc.) should never need to change for a reason that is not related to its central purpose.

In order to have a better understanding of this principle, let's consider a Tic Tac Toe program, written in Ruby. This program has a ```Board``` class:

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

Our ```Board``` has methods that generate, display and update a Tic Tac Toe board. At first glance, this class may appear to adhere to the Single Responsibility Principle without a problem; each of the three methods is specifically related to a ```Board``` object. However, let's look a little closer, and start to view this class in terms of *reasons for change*. We can see that both the ```generate_blank_board``` and ```update_board``` methods are dealing specifically with the **state** of the board itself. While, ```display_current_board``` deals with **displaying** the board to the user.

In her book Practical Object-Oriented Design in Ruby, Sandi Metz describes one of my favorite tips for writing software: when describing what a class (or any other software entity) does in one sentence, if you use the words "and" or "or", you're most likely breaking the Single Responsibility Principle. If you use the word "and", your class has at least two responsibilities. If you use the word "or", the class likely has more than one responsibility, and there's a good chance the responsibilities aren't related to one another.  Let's try this out on our ```Board``` class: It keeps track of a Tic Tac Toe board's state **and** displays this state to the user. Busted!

We've now determined that our ```Board``` class does in fact violate the Single Responsibility Principle, but why do we care?

Leaving these two separate responsibilities in the ```Board``` class, would essentially be coupling the idea of a ```Board```'s state with it's display. If we ever want to change how a board's **state is stored** or how a board is **displayed**, we would be dealing with two very different responsibilities.  Say our requirements change, and now instead of displaying our game in the terminal, we also want to display it in a browser as well. While creating a browser UI, there could be a chance that we could introduce a bug into the "state keeping" section of our class. Furthermore, since these two responsibilities are coupled into one class, there would also be a chance
that this one change breaks another part of our game system.

Let's dig back into our Tic Tac Toe example… As we mentioned above, it is quite possible that our Tic Tac Toe game evolves in such a way that we decide to extend our program to allow the user to play in a browser in addition to our initial display in the terminal.  How could we accomplish this? Our first option could be to use the same ```Board``` class that we defined above, and to add the new functionality to it:

```
class Board

  def generate_blank_board
    #generates an empty board state
  end

  def display_current_board_in_terminal
    #displays the board on the command line
  end

  def display_current_board_in_browser [addition -> highlight or make a
different color]
    #displays the current board in a browser
  end

  def update_board
    #updated the board's state
  end
end
```

Now we have a ```Board``` class that does exactly what we need it to do… or does it?  If we look at this code a little deeper, we'll realize that this class, which is responsible for rendering the game in the browser, shouldn't need to know anything about how the game is displayed in the terminal. But, the way that we've currently written ```Board```, the code which creates the browser’s user interface is very tightly coupled with the terminal’s user interface.

Let's look at how a browser game may utilize our extended ```Board``` class:

class BrowserGame
  def play_the_game
    board = Board.new
    board.display_current_board_in_browser

    #other game logic
  end
end

And a terminal game may look something like this:

class TerminalGame
  def play_the_game
    board = Board.new
    board.display_current_board_in_terminal

    #other game logic
  end
end

Every time we need to make a change to ```display_current_board_in_terminal``` for our ```TerminalGame```, we'll be touching the ```Board``` class and thereby risk introducing a bug or some sort of unintended side effect to any entity that uses ```Board```.  For example, since ```BrowserGame``` also uses ```Board``` this side effect could potentially trickle down and also effect how the game behaves in the browser as well, even though ```BrowserGame``` doesn't ever use ```display_current_board_in_terminal```!

Our other option to reuse ```Board``` in its current state would be to duplicate it, and create a ```BrowserBoard```, which has a display method that is specifically for ```BrowserGame``` (```display_current_board_in_browser```):

```
class BrowserBoard

  def generate_blank_board
    #generates an empty board state
  end

  def display_current_board_in_browser
    #displays the board to the browser
  end

  def update_board
    #updated the board's state
  end
end
```

This choice is flawed as well. If we ever need to change the way a ```Board``` behaves, like the logic it uses to update it’s state for example, we would need to change the ```update_board``` method in *both* ```BrowserBoard``` and our original terminal ```Board```.  This opens us up for double the chance of introducing a bug or inconsistency to our system. Plus, it is just tedious to update the same method twice!

So, how do we fix our original ```Board``` class above so that it adheres to SRP? Instead of including code to both **display** a board and **keep it's state** in one class, let's split those two pieces into separate classes. First we’ll pull the code dealing with the board’s state out into its own class, aptly named, ```BoardState```:

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

Then, for each new user interface that we want to incorporate into our application, we can simply create a new class to handle the presentation of the board specific to that outlet. So, for our ```BrowserGame```, we can create a ```BrowserBoardPresenter```:

```
class BrowserBoardPresenter
  def display_current_board
    #displays the board to the ui
  end
end
```

We can use this presenter in conjunction with our general purpose ```BoardState``` class to refactor our Tic Tac Toe game without changing the game’s overall behavior. We can instantiate a new ```BoardState```  and a new ```BoardPresenter``` and simply pass the board’s current state to the presenter every time we want to display it, in whichever outlet we choose!

board_state = BoardState.new
board_state.update_board

board_presenter = BoardPresenter.new
board_presenter.display_current_board(@board_state)

This is a fairly simple change, yet now we have added   a great deal of flexibility to our system in being able to add or change our user interface, without ever needing to touch our primary game logic.  Since we’re encapsulating the game’s display-centric code in our ```BrowserBoardPresenter```, we’re free to make any changes we want to
the presentation without fear of breaking something else! NEED TO FINISH
IT OFF


Sources:
Practical Object-Oriented Design in Ruby by Sandi Metz
Agile Software Development, Principles, Patterns, and Practices by
Robert "Uncle Bob" Martin

