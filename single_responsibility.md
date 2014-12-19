Single Responsibility Principle
===============================
One of my favorite tips for writing software comes from one of my favorite software role models, Sandi Metz. In her book Practical Object-Oriented Design in Ruby, she says that when describing what a class (or any other software entity) does in one sentence, if you use the words "and" or "or", you're most likely breaking the Single Responsibility Principle. If you use the word "and", your class has at least two responsibilities. If you use the word "or", the class likely has more than one responsibility, and there's a good chance the responsibilities aren't related to one another.  Let's try to desribe this ```Board``` class written for a Tic Tac Toe program (in Ruby):

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

How would we describe this class?  It keeps track of the board's state **and** displays this state to the user. It looks like we have a SRP violation! But, now that we've determined we have a violation, why should we care?

Including two separate responsibilities in one class, puts us in danger of
coupling pieces of functionality that have no reason to be dependent on one another. In the our ```Board```, we are coupling the ideas of a ```Board```'s state with its display. If we ever want to change how a board's **state is stored** or how a board is **displayed**, we would be dealing with two very different responsibilities.  Say our requirements change, and now instead of displaying our game in the terminal, we also want to display it in a browser as well. While creating a browser UI, there could be a chance that introduce a bug into the state keeping section of our class. 

Let's dig back into our Tic Tac Toe example.  How could we accomplish the display of the board in a browser? Our first option could be to use the same ```Board``` class that we defined above, and to add the new functionality to it:

```
class Board

  def generate_blank_board
    #generates an empty board state
  end

  def display_current_board_in_terminal
    #displays the board on the command line
  end

  def display_current_board_in_browser [-> highlight or make a
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

<!-- class BrowserGame
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
end -->

Every time we need to make a change to ```display_current_board_in_terminal``` for our terminal game, we'll be touching the ```Board``` class and thereby risk introducing a bug or some sort of unintended side effect to any entity that uses ```Board```.  For example, since our browser game also uses ```Board``` this side effect could potentially trickle down and also effect how the game behaves in the browser as well, even though it doesn't ever use ```display_current_board_in_terminal```!

Our other option to reuse ```Board``` in its current state may be to duplicate it, and create a ```BrowserBoard```, which has a display method that is specifically for the browser game (```display_current_board_in_browser```):

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

So, how do we fix our original ```Board``` class above so that it adheres to SRP? Instead of including code to both **display** a board and **keep it's state** in one class, let's split those two pieces of functionatliy into separate classes. First we’ll pull out the code dealing with the board’s state into its own class, aptly named, ```BoardState```:

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

Then, for each new user interface that we want to incorporate into our application, we can simply create a new class to handle the presentation of the board specific to that outlet. So, for our browser game, we can create a ```BrowserBoardPresenter```:

```
class BrowserBoardPresenter
  def display_current_board
    #displays the board to the ui
  end
end
```

We can use this presenter in conjunction with our general purpose ```BoardState``` class to refactor our Tic Tac Toe game without changing the game’s overall behavior. We'll instantiate a new ```BoardState```  and a new ```BoardPresenter``` and simply pass the board’s current state to the presenter every time we want to display it, in whichever outlet we choose!

board_state = BoardState.new
board_state.update_board

browser_board_presenter = BrowserBoardPresenter.new
browser_board_presenter.display_current_board(@board_state)

Our ```BoardPresenter``` will only ever need to change if and when we'd like the user interface in the browser to change, and these potential changes can be done so completely independently of how we keep track of our board's state. Likewise, our ```BoardState``` will only change when its state keeping logic needs to change, without effecting the board's presentation at all. By all accounts, it looks like we've corrected our SRP violation! But just to be sure, let's run these two classes through our handy "Sandi Metz SRP Checker", by answering the quesiton, 'What do each of these classes do?'. The ```BoardState``` class keeps the state of a Tic Tac Toe board. The ```BrowserBoardPresenter``` displays a Tic Tac Toe board in a browser. It looks like we've successfully elimitated any "ands" from each of our class's descriptions, and thereby are


Sources:

Practical Object-Oriented Design in Ruby by Sandi Metz

Agile Software Development, Principles, Patterns, and Practices by Robert "Uncle Bob" Martin

