# Mastermind (part 2)

In the second part we are working on the game, a player has 12 chances to break the code.

We are going to change the game engine and also add random drawing of the key code..

Here again we first show all the tests before implementing them, they are our todo list.



## The tests for the game




* game_engine_test.rb

```ruby
class PlayerGuessedServiceTest < Test::Unit::TestCase

  def setup
    @engine = GameEngine.new('GRWR')
  end
  
  def test_11_wins_immediately
    assert_equal :win, @engine.player_guessed('GRWR')
  end
  
  def test_12_wins_after_some_guesses
    
    assert_equal [], @engine.player_guessed('BBBB')
    assert_equal [:black], @engine.player_guessed('GBBB')
    assert_equal :win, @engine.player_guessed('GRWR')
    
  end
  
  def test_13_defeats
    
    number_of_guesses_before_last = GameEngine::MAX_GUESSES - 1
    
    number_of_guesses_before_last.times do
      @engine.player_guessed('BBBB')
    end
  
    assert_equal :defeat, @engine.player_guessed('BBBB')
    
  end
  
  def test_14_wins_on_last_chance
    
    number_of_guesses_before_last = GameEngine::MAX_GUESSES - 1
    
    number_of_guesses_before_last.times do
      @engine.player_guessed('BBBB')
    end
  
    assert_equal :win, @engine.player_guessed('GRWR')
    
  end
  
end

```

The tests expect a new API from the game engine, a `player_guessed` method. The method either returns an array of `white` and `black` pegs (_symbols_), either returns symbols named `defeat` or `win`.

Tests also expect `MAX_GUESSES` to provide how many guesses a player can make before losing.

Notice how the code simulates invalid guesses to reach the edge cases, on line 23, for instance, the code repeats an invalid guess `MAX_GUESSES - 1` times so that the guess on line 27 is the last one.

We build on the current engine implementation presented in the first part of the series. We are not going to show the whole code, we focus on the `player_guessed` method.

### Test: the player wins immediately

Our first test is about a player winning at the very first guess.



* game_engine.rb

```ruby
  # returns an array of :white and :black values, :win or :defeat
  def player_guessed guess
  
    result = check(guess)
    
    return :win if result == [:black, :black, :black, :black]
    
    result
    
  end

```

On line 4 we reuse the `check` method. On line 6, we check if it returns four `black` symbols, meaning the guess was successful.

<pre class='console'>
$&gt; ruby  game_engine_test.rb -n /11/
Loaded suite /tmp/release/mastermind/game_engine_test
Started
.

Finished in 0.000352266 seconds.

1 tests, 1 assertions, 0 failures, 0 errors, 0 pendings, 0 omissions, 0 notifications
100% passed

2838.76 tests/s, 2838.76 assertions/s
</pre>

### Test: the player wins after some guesses

As long as we don't add code that implements the _defeat aspects_, the current version is fine...

<pre class='console'>
$&gt; ruby  game_engine_test.rb -n /1[12]/
Loaded suite /tmp/release/mastermind/game_engine_test
Started
..

Finished in 0.000502276 seconds.

2 tests, 4 assertions, 0 failures, 0 errors, 0 pendings, 0 omissions, 0 notifications
100% passed

3981.87 tests/s, 7963.75 assertions/s
</pre>

### Test: the player defeats

A player defeats if he/she does not break the key code after 12 tries. We set up a constant equal to 12.


* game_engine.rb

```ruby
  MAX_GUESSES = 12

```

We need to count the number of guesses, therefore we add the `count_guesses` attribute initialized to 0 (see line 3).



* game_engine.rb

```ruby
  def initialize key_code
    @key_code = key_code.upcase
    @count_guesses = 0
  end

```

In the `player_guessed` method we start checking if the player didn't exceed the allowed number of guesses.



* game_engine.rb

```ruby
  # returns an array of :white and :black values, :win or :defeat
  def player_guessed guess

    return :defeat if @count_guesses > MAX_GUESSES
    
    @count_guesses = @count_guesses + 1
  
    result = check(guess)
    
    return :win if result == [:black, :black, :black, :black]
    
    result
    
  end

```

Line 4 returns the `defeat` symbol if the counter is greater than the maximum allowed so that once the guesses exceed the maximum it will always fail.

Line 6 increments the counter, as we are now evaluating a new guess.

We call the `check` method, on line 8, and if the guess was successful the code returns `win` (line 10). Otherwise before returning the feedback, we need to check if the current guess is not the last one.



* game_engine.rb

```ruby
  # returns an array of :white and :black values, :win or :defeat
  def player_guessed guess

    return :defeat if @count_guesses > MAX_GUESSES
    
    @count_guesses = @count_guesses + 1
  
    result = check(guess)
    
    return :win if result == [:black, :black, :black, :black]
    return :defeat if @count_guesses == MAX_GUESSES
    
    result
    
  end

```

Line 11, checks if the player did reach the last guess.

<pre class='console'>
$&gt; ruby  game_engine_test.rb -n /1[123]/
Loaded suite /tmp/release/mastermind/game_engine_test
Started
...

Finished in 0.000621104 seconds.

3 tests, 5 assertions, 0 failures, 0 errors, 0 pendings, 0 omissions, 0 notifications
100% passed

4830.11 tests/s, 8050.18 assertions/s
</pre>

### Test: the player wins at the last moment

The last test and all the test do pass...

<pre class='console'>
$&gt; ruby  game_engine_test.rb
Loaded suite /tmp/release/mastermind/game_engine_test
Started
...............

Finished in 0.001599207 seconds.

15 tests, 17 assertions, 0 failures, 0 errors, 0 pendings, 0 omissions, 0 notifications
100% passed

9379.65 tests/s, 10630.27 assertions/s
</pre>

## Let's play then...

Now that the engine is ready, we must make it usable by a real user.

A very simple implementation is going to run in the terminal.


* game_textui.rb

```ruby
require 'game_engine'  

def play_game
  
  engine = GameEngine.new

  loop do

    print "Make a guess (or exit): "
    guess = gets.chomp
    
    exit if guess == 'exit'

    result = engine.player_guessed(guess)

    if result == :win
      puts "You win!"
      break
    elsif result == :loose
      puts "You loose, the code was '#{engine.key_code}'"
      break
    end
    
    p result
    
  end
  
end

loop do

    play_game

    print "Do you want to play again (yes or no)? "
    answer = gets.chomp
    
    break if answer == 'no'

end

```

We first require the game engine, then define a method named `play_game`.

At line 32 we start a loop that first calls `play_game` (ensuring to play one game). Then (line 34), the code asks the user if he/she wants to play again.

The `play_game` method is pretty simple, it prompts the user for a guess. Uses the engine to check the guess (line 14) and give a feedback to the user depending on the check result.

## I always win!

The problem with our implementation is that the game engine uses only one code. We must introduce the random drawing of the code (without breaking the tests).

Because the tests need the _hardcoded_ code while the interactive game needs a random code, we introduce the _code drawer_ concept.


* code_drawer.rb

```ruby
class CodeDrawer
  
  COLORS = ['W', 'R', 'B', 'G', 'Y', 'P', 'O', 'G']
  
  # returns a random 4-color code
  def draw
    COLORS[rand(COLORS.length)] +
    COLORS[rand(COLORS.length)] +
    COLORS[rand(COLORS.length)] +
    COLORS[rand(COLORS.length)]
  end
  
end

```

The creation of a `GameEngine` requires a code drawer which by default is the real `CodeDrawer`.

In the tests we create a mock version of the code drawer like this.





* game_engine_test.rb

```ruby
class CodeDrawerMocker

  def initialize code
    @code = code
  end
  
  # returns a random 4-color code
  def draw
    @code
  end
  
end

class PlayerGuessedServiceTest < Test::Unit::TestCase

  def setup
    @engine = GameEngine.new(CodeDrawerMocker.new('GRWR'))
  end
  
  def test_11_wins_immediately
    assert_equal :win, @engine.player_guessed('GRWR')
  end
  
  def test_12_wins_after_some_guesses
    
    assert_equal [], @engine.player_guessed('BBBB')
    assert_equal [:black], @engine.player_guessed('GBBB')
    assert_equal :win, @engine.player_guessed('GRWR')
    
  end
  
  def test_13_defeats
    
    number_of_guesses_before_last = GameEngine::MAX_GUESSES - 1
    
    number_of_guesses_before_last.times do
      @engine.player_guessed('BBBB')
    end
  
    assert_equal :defeat, @engine.player_guessed('BBBB')
    
  end
  
  def test_14_wins_on_last_chance
    
    number_of_guesses_before_last = GameEngine::MAX_GUESSES - 1
    
    number_of_guesses_before_last.times do
      @engine.player_guessed('BBBB')
    end
  
    assert_equal :win, @engine.player_guessed('GRWR')
    
  end
  
end

```

We define the mock class (line 1-12) and use it at line 17 (we apply the same change to the previous test cases setup).

And we also need to change `GameEngine` initialization.


* game_engine.rb

```ruby
  def initialize drawer = CodeDrawer.new
    
    @drawer = drawer
    
    @count_guesses = 0
    @key_code = @drawer.draw.upcase
    
  end

```

Let's run the test a last time.

<pre class='console'>
$&gt; ruby  game_engine_test.rb
Loaded suite /tmp/release/mastermind/game_engine_test
Started
...............

Finished in 0.001652098 seconds.

15 tests, 17 assertions, 0 failures, 0 errors, 0 pendings, 0 omissions, 0 notifications
100% passed

9079.36 tests/s, 10289.95 assertions/s
</pre>


_Jean Lazarou  (Feb 28, 2016 - 08:59AM)_
