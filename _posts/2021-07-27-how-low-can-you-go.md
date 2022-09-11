---
layout: article
title: How low can you go?
tags: python, fun, game
mode: immersive
header:
  theme: dark
article_header:
  type: overlay
  theme: dark
  background_color: '#203028'
  background_image:
    src: https://wallpaperaccess.com/full/3329944.jpg
---


When it comes to having fun at work, simple games like rock, paper, scissors often get pulled out to put a twist on making decisions. Burrito or gyros for lunch? Who gets the ticket to tonights big game - _go Celtics_ - offered up by a colleague who can't make it? Today I am gonna talk about my favorite such game and we will also implement it in python. As of the writing of this article I am using python `3.8`.


🎱 The game
-----------

The name of the game is _How low can you go_? Can be played by an arbitrary number of players that makes it suitable to be played in an office setting. The rules are dead simple: the lowest positive unique integer entry wins.
Let's digest that sentence a bit. 
- _Integer entry_: the game is played by picking numbers
- _Lowest positive_: the lowest number you can pick is 1
- _Unique_: two players picking the same number get eliminated

What I love about this game is that the rules are so simple that first time players usually skimp over the details, and haphazardly pick 1 as their entry and end up losing. See, you have to the biggest emphasis is on unique. If two or more players pick 1 as their entry, they all get eliminated. Picking a winner would continue to the next lowest entries and continue up until the first unique entry is found. What is especially funny about this game is that technically you can be the winner with an entry that does not fit into the category of low at all. You could enter the googol number - that is 1 followed by 100 zeros - and still be the winner, if all other players eliminated each other by picking shared numbers as their entries.

▶️ Example round
---------------

Let's say there are 5 players. Each of them pick 1 random positive number. Player one picks 3, player two picks 1, player three picks 1, player four picks 2, player five picks 3. Player two and three picked the lowest numbers, however, their entries are not unique, so they eliminated each other. Player four picked 2 and no other player picked it that makes player four the winner. Players one and five both picked 3. Should player four picked a larger number, they would have eliminated each other.


🕹 Code
-------

Let's try to code this simple game in python. As you will soon see, this makes a perfect pair coding problem in a job interview setting because the logic is very simple, yet there are some nuances that still make it an interesting coding challenge. 

Let's start with some scaffolding. To keep it simple this will be a console game, making the user interface text based. Now, choosing the console does not mean that the app does not need to consider basic [human interface guidelines](https://developer.apple.com/design/human-interface-guidelines/patterns/onboarding). To the contrary, the amount of thought one puts into the applications user interface can easily make or break the end result. 

```python
    def how_low_can_you_go(intro: bool):
        print('How low can you go')
        print('------------------')
        if intro:
          print("How to play: ")
          print("Lowest positive unique integer entry wins.")
```

The method will be the entry point for the game. The game welcomes the player by printing the name of the game. Setting the boolean `intro` argument `true`, the game also prints a basic help on how to play. The flag comes in handy for returning players. They have played before, so seeing the rules again might not be of much use to them anymore. We will make use of this flag at the very end, where we ask the players if they would like to play another round. The game tends to be addictive.

👾 Ready Player One
-------------------

In the first iteration, we will implement a single player game vs. the computer. This allows us to keep it super simple while also put some rudimentary artificial intelligence in place to make it not so easy but fun to play.

First, we have to figure out how many players there will be. _n_ being the number of player, _n-1_ players will be played by the computer and the human player will be player _n_. Input and output of console applications tend to be boilerplate so it makes sense to look for an existing solution. My goto choice in python is the [Click](https://click.palletsprojects.com/en/8.1.x/) library. The name can be a bit confusing at first, as the mouse is usually not at hand in a console setting. However, once you learn that click is an acronym that stands for _Command Line Interface Creation Kit_ it all makes good sense.

We prompt for the number of players and let click handle the chore of validating user input. Zero or negative number of players does not make much sense, ain't that true?

```python
  import click

    ...
    num_players = click.prompt("Number of players?", type=click.IntRange(min=1))
```

Once we know how many players there are, we need to collect the guesses of the computer players. A simple array will do the trick.

```python
  from time import sleep

    ...
    guesses = []
    collect_guesses(num_players, guesses)
```

Method `collect_guesses` is where things start to get interesting. Following HIG, it makes sense to distinguish the case of 2 players vs. more players. For the 2 player case, there is only one computer player, whereas for more than two players, there are more then two computer players. For every computer player, we call the `create_guess_tuple` method that will generate a random entry for a player. For more than two players, we can spice up the user experience a bit by creating the illusion that the computer players are thinking super deep when they are picking their entries. To achieve this, we rely on the `sleep` method of the built in `time` package and the `progressbar` feature of click. We simply sleep a little for every computer player and show a progress bar in the meantime.

```python
 def collect_guesses(num_players, guesses):
    if num_players == 2: 
        guesses.append(create_guess_tuple(1))
        print(f"Player #1 have made their guess.")
    elif num_players > 2:
        with click.progressbar(range(1, num_players), label=f"Players #1-#{num_players - 1} guessing", show_eta=False, show_percent=False) as players:
            for player in players:
                guesses.append(create_guess_tuple(player))
                sleep(3 / num_players)
        print(f"Players #1-#{num_players - 1} have made their guesses.")
    print(f"You are player #{num_players}.")
```

The central point of the game logic is implemented in method `create_guess_tuple`. The first observation we can make is that we need to generate a random number. However, to make the game more challenging to play, any random number will not do. We would like to achieve a distribution that is skewed towards lower numbers. Lucky for us, we chose python as our language for this exercise and it comes with third party packages to make solving any problem a breeze. Packages like [NumPy](https://numpy.org/) certainly has skew functions that we could use here. However, NumPy is not the lightest of packages so at the same time it feels wrong to whip out a big gun for such a small thing as out little game at hand. Instead, we will achieve the skewing with a clever little trick. We will make the computer players play a different strategy using their indexes. Player 1 will pick a random number from smaller set of numbers, than player 2, 3 and so on. This simple logic will guarantee that some computer players will always pick low numbers close to 1. At the same time, we also make all computer players follow a different strategy that helps us avoid the situation of computer players constantly eliminating each other. Using the constant of `GUESS_MULTIPLIER` we can control how tight the computer players pick their entries. E.g.: 3 computer players, with a constant of 4. Computer player 1 will pick a random entry from the set of (1, 4). Computer player 2 will pick a random entry from the set of (1, 8). Computer player 1 will pick a random entry from the set of (1, 12). As you see, the chance of 3 computer players picking entries from 1 to 4 is triple that of numbers larger than 4, thus we have achieved skewing without having to rely on complex math and heavy external libraries.

```python
  from random import randint
  GUESS_MULTIPLIER = 4

  def create_guess_tuple(player):
      return (player, randint(1, player * GUESS_MULTIPLIER))
```

Once computer players made their guesses, it is time to collect the human player's entry:

```python
  human_guess = click.prompt("Your guess", type=click.IntRange(min=1))
  human_player = (num_players, human_guess)
  guesses.append(human_player)
```

By this point, we have all the entries collected in the array `guesses`. All we have to do now is to evaluate the entries and pick a winner. First, we sort the entries ascending and offload the chore of picking the winner to method `evaluate_winner`.

```python
  guesses.sort(key = lambda x: x[1])
  winner = evaluate_winner(guesses)
```

To evaluate the winner, we have to consider all pairs of entries. For this we make use of the `combinations` function of the built-in `itertools` package, passing `2` as the second argument. We also rely on the power of a `Set` data structure to collect unique guesses and see if the entry under consideration has already been entered. For some reason people new to the software engineering game tend to have more difficulty grasping the power of the set data structure. Given its similarities with a `List` both falling under the category of collections, they tend to use `List` as their silver bullet when in need of a collection data structure. A `Set` by definition can only contain a certain element once, so if we pass in a list of `Set([1,2,2,1,1,2])` to a set constructor, the resulting set will be `{1,2}` thus achieving uniqueness. There is a big difference in the time complexity of lookup operations like `contains` when it comes to `Set` vs `Array`. The former can do the trick in constant time, making it the default choice when it comes to solving for the problem of 'is this element a member of this collection'?

```python
  from itertools import combinations

  def evaluate_winner(guesses):
      unique_guesses = set()
      winner = None
      for (guess_left, guess_right) in for pair in combinations(guesses,2):
          if guess_left[1] != guess_right[1] and not guess_left[1] in unique_guesses:
              winner = guess_left
              break
          unique_guesses.add(guess_left[1])
      if not winner and not guesses[-1][1] in unique_guesses: winner = guesses[-1]
      return winner
```

Once we have a winner, all that is left is to make a big announcement. The final nuance of the game that I have not mentioned yet is that in rare circumstances all players can be eliminated in which case the game is a draw.

```python
    ...
    show_result(guesses, winner, human_player)

    def show_result(guesses, winner, human):
        print("")
        for guess in guesses:
            win = guess == winner
            result = "wins" if win else "lost"
            print(f"Player #{'{: >2}'.format(guess[0]) if len(guesses) > 9 else guess[0]} guess of {'{: >2}'.format(guess[1])} {result}")
        if not winner:
            print("Game is a rare draw")
```

Finally, to cater for the game addicts and to frame my article with what I hinted at in the beginning we can offer the player to play one more round. 

```python
    if click.confirm("Play another round?")
       how_low_can_you_go(false)
```

Happy coding!
