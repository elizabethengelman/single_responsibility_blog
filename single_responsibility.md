Single Responsibility Principle
===============================

What is the Single Responsibility Principle? And why is it important?

The SRP is first of Uncle Bob's SOLID principles, and is perhaps one of
the most important principles [DIFFERENT WORD FOR PRINCIPLES] to follow when writing good, clean
software. By adhering to the SOLID principles, we can set ourselves up
for success, in that we're creating code that is more easily maintaned,
and extended as our project evolves. In a nutshell, the SRP states that
each software entity that we create should have one, and only one reason for changing.

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
    #generate a board state
  end

  def display_current_board
    #displays the board to the ui
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

Leaving these two separate responsibilities in the ```Board``` module,
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
one of my favorite software tips: when describing what a class (or any
other software entity) does in one sentence, if you use the words "and" or "or", you're most
likely breaking the Single Responsibility Principle. If you use the
word "and" your class has at least two responsibiliites. If you use the
word "or" the class likely has more than one responsibility, and there's
a good chance the two responsibilities aren't related to one another.
Let's try this out on our ```Board``` class: It keeps track of a Tic Tac
Toe board's state **and** displays this state to the user. Busted!

We've now determined that our ```Board``` class does in fact
violate the Single Responsibility Principle, but why do we care?

When we adhere to the Single Responsibility Principle, we're on the road
to ensuring [DIFFERENT WORD FOR ENSURE] that our code is reusable.
A benefit of creating code that is reusable is that we're able to reduce
our code duplication, and as we know, crafting software that is DRY is
yet another important [DIFFERENT WORD] tenant to follow when creating
software. By





Another essential tenant involved in creating software that is maintainable
and extendable is code reusibility. The benefit of creating code that

Another important implication of the Single Responsibility Principle is
the idea of code reusability. Let's say that we had use for a module
that was only concerned with *displaying* a game board. In order to use
```Board``` as it currently is designed, we would also have to include
all the logic involved in generating and updating the board's state,
even though this particular application doesn't care about those
functions. This would include a lot of unused code, and would force our
new application to retest and redeploy our game system any time there
was a change to how the ```Board``` state was updated - though that
isn't even used in this new application.

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
