# Classroom UNO Game
This repository includes a simple python program to simulate a classroom game based on the card game UNO.
The concept for the game is due to [Monica Genta's podcast](https://monicagenta.com/podcast/uno-classroom-strategy-that-will-make-your-classroom-wild-in-a-good-way/).

# Examples
Multiple examples are provided at the bottom of the ``classroom_uno_game.py`` file.

## Single round
The ``UnoGame`` class can be used to simulate a single round of the game,
```py
game = UnoGame("test", verbose=True)
game.play()
```
will print the results to the console,
```
3 card
test - Round 1 ended with, 
	 * cards drawn this round: 1
	 * points = 47
	 * points to distribute = 0
```
In this case the player "test" started with 50 points (default) and drew one 3 card which brought their score to 47.

## Playing an entire game
The following code simulates an entire game,
```py
game = UnoGame("test", verbose=True)
while not game.game_over:
    game.play()
```
With ``verbose=True`` the console will output the results of each round.

## Group play
The ``group_play`` function is also included outside of the ``UnoGame`` class to simulate multi-classroom games.
The game contains adversarial mechanics where teams can give points to others.
By default, other players will give points to the players with the lowest score (the winning team).

Example usage of the group play function,
```py
names = ["Period A", "Period B", "Period C", "Homeroom"]
players = group_play(*names,verbose=True)
```
The ``UnoGame`` objects are returned as the list ``players``.
With ``verbose=True`` the console will print the back-and-forth between rounds.
For example, 
```
Group Play: Round 0
Period A's turn:
	start with 50 points.
	end with 46 points.
Period B's turn:
	start with 50 points.
	give 8 point(s) to Period A!
	end with 50 points.
Period C's turn:
	start with 50 points.
	end with 47 points.
Homeroom's turn:
	start with 50 points.
	give 2 point(s) to Period A!
	end with 28 points.
Group Play: Round 1
Period A's turn:
	start with 56 points.
	give 4 point(s) to Homeroom!
	end with 43 points.
Period B's turn:
	start with 50 points.
	end with 46 points.
Period C's turn:
	start with 47 points.
	end with 46 points.
Homeroom's turn:
	start with 32 points.
	give 2 point(s) to Period A!
	end with 32 points.
.
.
.
```
In this example, Period A started off by reducing their score to 46 points (making them the leader).
Period B drew a reverse-card and, because Period A was in the lead, Period B gave 8 points to Period A!
Therefore, Period A starts Round 1 with 56 points.

The following code can be used to plot the points at the end of each round for group play.
```py
fig, ax = plt.subplots(1,1, num="UnoGame: Adversarial Gameplay")
for p in players:
    ax.plot(p.points_per_round, '-o', label=f"{p.name}")
ax.axhline(1,color="k",ls="--")
ax.set_xlabel("Rounds")
ax.set_ylabel("Points")
plt.legend()    
```
The result is,
<p align="center">
  <img width="500" alt="image" src="https://github.com/quinntpratt/classroom_uno_game/assets/32646393/c03ad22b-0de0-4ca5-9d96-790e4bc007b4">
</p>

## Monte-Carlo analysis
Finally, the code can be used to simulate thousands of games and determine the average number of rounds played for 'peaceful' and 'adversarial' gameplay.
For example, for peaceful gameplay one may use the code,
```py
N_games = 1000
rounds = np.zeros(N_games)
cards_drawn = np.zeros(N_games)
for i in range(N_games):
    game = UnoGame("MC")
    while not game.game_over:
        game.play()
    rounds[i] = game.round
    cards_drawn[i] = game.n_drawn_total

fig, (axr, axc) = plt.subplots(1,2,num="UnoGame: MonteCarlo Analysis")
axr.hist(rounds, ec="k",color="b",alpha=0.3,label=f"Peaceful (N={N_games})")
axr.set_xlabel("Rounds Played")
axc.hist(cards_drawn, ec="k",color="b",alpha=0.3,label=f"Peaceful (N={N_games})")
axc.set_xlabel("Cards Drawn")
```
The resulting plot is,
<p align="center">
  <img width="700" alt="image" src="https://github.com/quinntpratt/classroom_uno_game/assets/32646393/129ad8f1-10fa-4d47-83f2-6b041583be96">
</p>
Starting with 50 points, if students are allowed to draw 1 card per round... this analysis suggests they will play an average of 10.8 rounds to finish the game and they will draw an average of 14.6 cards over the course of the game.

We also can compare histograms between peaceful and allowing adversarial play.
The following code can be used,
```py
N_games = 250
N_players = 4
names = [f"Player {i}" for i in range(N_players)]
rounds = np.zeros((N_games,N_players))
cards_drawn = np.zeros((N_games,N_players))
for i in range(N_games):
    players = group_play(*names)
    rounds[i,:] = np.array([p.round for p in players])
    cards_drawn[i,:] = np.array([p.n_drawn_total for p in players])

axr.hist(rounds.flatten(), ec="k",color="r",alpha=0.3,label=f"Adversarial (N={N_games},{N_players} players)")
axc.hist(cards_drawn.flatten(), ec="k",color="r",alpha=0.3,label=f"Adversarial (N={N_games},{N_players} players)")
axr.legend()
axc.legend()
```
The result of comparing histograms in this case (50 points starting, 1 card drawn per round) indicates that adversarial play only slightly lengthens the game.
On average it took 11.6 rounds to end the game with adversarial play compared to 10.6 with peaceful play.
<p align="center">
  <img width="700" alt="image" src="https://github.com/quinntpratt/classroom_uno_game/assets/32646393/b70ecf4f-f489-478d-9a55-50d1ee6c8583">
</p>

