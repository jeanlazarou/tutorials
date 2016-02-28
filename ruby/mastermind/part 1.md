# Mastermind

<style>
.pub-sidebar {
  width: 500px;
  font-size: 14px;
  margin-left: auto ;
  margin-right: auto ;
  padding: 20px;
  background: none repeat scroll 0% 0% #EEEEDD;
  border: 1px solid #DBDBCC;
  box-shadow: 3px 3px 5px #999;
  border-radius: 0.7em;
}
</style>

This article is the first part of a TDD (test driven development) example in Ruby. It implements a game named _Mastermind_.

We are going to split the tests in two parts: _the tests for the rules_ and _the tests for the game_. We show all the tests first so that the reader can implement them before reading the solution presented here.

We are going to implement the tests in the order in which they appear, leading to refactorings and fixes, when the following tests fail.

The second part ends with an implementation of a simple _text user interface_ (text UI).



## The game

The player must find a key code, he/she makes guesses and receives a feedback. He/she has a limit of 12 guesses otherwise he/she loses. The key code contains 4 colours out of 8 possible colours. As feedback, the user receives white or black _pegs_. Each white peg means that the guess has one correct colour but is not at the right place. Each black peg means the guess has one correct colour at the right place. The player does not know which colour is the correct one.

As an example, let the key code be `RBGW` (red, blue, green and white). If the player guesses `RWOY` (red, white, orange and yellow), he/she receives one white and one black peg.

## The tests for the rules

> Test names contain a number, so that we can use patterns to run part of the tests<br>
> (regular expression pattern with the -n option).
>
> If we wanted to run the first three tests we would write:
>
>  `ruby game_engine_test.rb -n /test_0[123]/`

We made a decision about the implementation, we use something we call a `GameEngine` which knows everything about the rules but does not interact with the user, it is an API for the UI code.




* game_engine_test.rb

```ruby
require 'test/unit'

require_relative 'game_engine'

class GameEngineTests < Test::Unit::TestCase

  def setup
    @engine = GameEngine.new('GRWR')
  end
  
  def test_01_totally_wrong_pattern
    assert_equal [], @engine.check('BOYB')
  end
  
  def test_02_correct_key_code
    assert_equal [:black, :black, :black, :black], @engine.check('GRWR')
  end
  
  def test_03_one_correct_color
    assert_equal [:white], @engine.check('ROYB')
  end
  
  def test_04_two_correct_colors
    assert_equal [:white, :white], @engine.check('ROYG')
  end
  
  def test_05_three_correct_colors    
    assert_equal [:white, :white, :white], @engine.check('RWYG')
  end
  
  def test_06_one_correct_color_and_right_position
    assert_equal [:black], @engine.check('GOYB')
  end
  
  def test_07_three_correct_colors_with_one_at_right_position
    assert_equal [:white, :white, :black], @engine.check('RYWG')
  end
  
  def test_08_engine_is_not_case_sensitive
    assert_equal [:white, :white, :black], @engine.check('rywg')
  end
  
  def test_09_only_4_character_codes_are_valid
    assert_equal [], @engine.check('rywgYY')
  end
  
  def test_10_repeating_same_color_produces_correct_feedback
    assert_equal [:black], @engine.check('GGGG')
  end
  
end

class GameEngineIsNotCaseSensitiveTest < Test::Unit::TestCase
  
  def test_engine_is_not_case_sensitive
    
    @engine = GameEngine.new('GRWR')
    
    assert_equal [:white, :white, :black], @engine.check('rywg')
    
  end
  
end


```

In the tests above we see that we need a `GameEngine` class which needs the (hidden) key code to create new instances, as shown hereafter.




* game_engine.rb

```ruby
class GameEngine

  def initialize key_code
    @key_code = key_code
  end
  
end

```

The APIs we imagined in the tests expect the class to provide a `check` method. It checks if the given pattern, a guess, is correct.


* game_engine.rb

```ruby
class GameEngine

  def initialize key_code
    @key_code = key_code
  end
  
  # returns an array of :white and :black values, with :white(s) first
  def check pattern
  end

end

```

Let's start making the tests pass...

### Test: guessing a totally wrong pattern

When a guess is totally wrong we receive an empty array as a result. All we need to do is return an empty array.


* game_engine.rb

```ruby
class GameEngine

  def initialize key_code
    @key_code = key_code
  end
  
  # returns an array of :white and :black values, with :white(s) first
  def check pattern
    []
  end

end

```

Run the test and see it passes. (we use the `-n` option of unit test command line to limit the test run to the first test)

<pre class='console'>
$&gt; ruby  game_engine_test.rb -n test_01_totally_wrong_pattern
Loaded suite /tmp/release/mastermind/game_engine_test
Started
.

Finished in 0.000364667 seconds.

1 tests, 1 assertions, 0 failures, 0 errors, 0 pendings, 0 omissions, 0 notifications
100% passed

2742.23 tests/s, 2742.23 assertions/s
</pre>

Our first success is of course useless, it reflects the usual: implement just enough to make the test pass...

### Test: correct key code

The easier is to test if the _pattern_ is equal to the hidden key code.


* game_engine.rb

```ruby
class GameEngine

  def initialize key_code
    @key_code = key_code
  end
  
  # returns an array of :white and :black values, with :white(s) first
  def check pattern
    if pattern == @key_code
      [:black, :black, :black, :black]
    end
  end

end

```

We must add the _else_ branch, which returns an empty array so that the first test still passes.


* game_engine.rb

```ruby
class GameEngine

  def initialize key_code
    @key_code = key_code
  end
  
  # returns an array of :white and :black values, with :white(s) first
  def check pattern
    if pattern == @key_code
      [:black, :black, :black, :black]
    else
      []
    end
  end

end

```

Run the two tests.

<pre class='console'>
$&gt; ruby  game_engine_test.rb -n /test_0[12]/
Loaded suite /tmp/release/mastermind/game_engine_test
Started
..

Finished in 0.000397565 seconds.

2 tests, 2 assertions, 0 failures, 0 errors, 0 pendings, 0 omissions, 0 notifications
100% passed

5030.62 tests/s, 5030.62 assertions/s
</pre>

### Test: one correct color

The first idea that came to mind was to play with strings, as the key code is a `String` object.

The `String` class has a method named `tr`, translate, which takes two `String` parameters. The first gives the list of characters we want to translate and the second gives the replacement characters. So, we could replace all the characters from the key code that appear in the guessed pattern with nothing and keep the invalid colours. If we know how many invalid colours the pattern does contain, we can compute how many valid colours it contains.
 
 

* game_engine.rb

```ruby
class GameEngine

  def initialize key_code
    @key_code = key_code
  end
  
  # returns an array of :white and :black values, with :white(s) first
  def check pattern
    if pattern == @key_code
      [:black, :black, :black, :black]
    else
      translated = pattern.tr(@key_code, '')
      pegs = [:white] * (pattern.length - translated.length)
    end
  end

end

```

As the current test only checks the behaviour of the engine for a valid colour, not for a correct position, the above implementation is the simplest that makes the test pass.

Line 13, generates an array containing `:white` symbols of the computed size: _the length of the given pattern (guessed colours) minus the length of the invalid colours_.

<pre class='console'>
$&gt; ruby  game_engine_test.rb -n test_03_one_correct_color
Loaded suite /tmp/release/mastermind/game_engine_test
Started
.

Finished in 0.00034856 seconds.

1 tests, 1 assertions, 0 failures, 0 errors, 0 pendings, 0 omissions, 0 notifications
100% passed

2868.95 tests/s, 2868.95 assertions/s
</pre>

The test passes, but the name `translated` we used on line 13 is not clear, a better name would be `invalid_colors`. Let's refactor the code.


* game_engine.rb

```ruby
class GameEngine

  def initialize key_code
    @key_code = key_code
  end
  
  # returns an array of :white and :black values, with :white(s) first
  def check pattern
    if pattern == @key_code
      [:black, :black, :black, :black]
    else
      invalid_colors = pattern.tr(@key_code, '')
      pegs = [:white] * (pattern.length - invalid_colors.length)
    end
  end

end

```


Run the first 3 tests to check we didn't break them.

<pre class='console'>
$&gt; ruby  game_engine_test.rb -n /test_0[1-3]/
Loaded suite /tmp/release/mastermind/game_engine_test
Started
...

Finished in 0.000547589 seconds.

3 tests, 3 assertions, 0 failures, 0 errors, 0 pendings, 0 omissions, 0 notifications
100% passed

5478.56 tests/s, 5478.56 assertions/s
</pre>

### Tests: two or three correct colors

We don't need to change the current code for the two tests...

<pre class='console'>
$&gt; ruby  game_engine_test.rb -n /test_0[1-5]/
Loaded suite /tmp/release/mastermind/game_engine_test
Started
.....

Finished in 0.000624504 seconds.

5 tests, 5 assertions, 0 failures, 0 errors, 0 pendings, 0 omissions, 0 notifications
100% passed

8006.35 tests/s, 8006.35 assertions/s
</pre>

### Test: one correct color and right position

The code returns only `:white` symbols even if the colours are at a correct position. We must check for correct positions, remove one `:white` symbol for each correct position and add one `:black` symbol.

As we only have four colours we can simply use comparisons.


* game_engine.rb

```ruby
class GameEngine

  def initialize key_code
    @key_code = key_code
  end
  
  # returns an array of :white and :black values, with :white(s) first
  def check pattern
    if pattern == @key_code
      [:black, :black, :black, :black]
    else
      invalid_colors = pattern.tr(@key_code, '')
      pegs = [:white] * (pattern.length - invalid_colors.length)

      if @key_code[0] == pattern[0]
        pegs.shift
        pegs << :black
      end

      if @key_code[1] == pattern[1]
        pegs.shift
        pegs << :black
      end

      if @key_code[2] == pattern[2]
        pegs.shift
        pegs << :black
      end

      if @key_code[3] == pattern[3]
        pegs.shift
        pegs << :black
      end

      pegs      

    end
  end

end

```

We repeat four times the same code pattern: compare the first character of the key code with the first character of the given pattern (line 15), if they match we remove one _white peg_ (line 16) and add a black peg (line 17). We repeat the same for the next three characters (colours).

<pre class='console'>
$&gt; ruby  game_engine_test.rb -n /test_0[1-6]/
Loaded suite /tmp/release/mastermind/game_engine_test
Started
......

Finished in 0.000702867 seconds.

6 tests, 6 assertions, 0 failures, 0 errors, 0 pendings, 0 omissions, 0 notifications
100% passed

8536.47 tests/s, 8536.47 assertions/s
</pre>

### Test: three correct colors with one at right position

We are at the 7th test which should pass...

<pre class='console'>
$&gt; ruby  game_engine_test.rb -n /test_0[1-7]/
Loaded suite /tmp/release/mastermind/game_engine_test
Started
.......

Finished in 0.000817402 seconds.

7 tests, 7 assertions, 0 failures, 0 errors, 0 pendings, 0 omissions, 0 notifications
100% passed

8563.72 tests/s, 8563.72 assertions/s
</pre>

### Test: engine is not case sensitive

What does _engine is not case sensitive_ mean?

As we use strings to represent colour sequences, we would like to have upper case letters to be equal to lower case letters. For instance, we would like `R`, meaning red, to be equal to `r`.

An easy way to resolve this case is to work with upper case letters. We can first force conversion of the given pattern to upper case letters.


* game_engine.rb

```ruby
class GameEngine

  def initialize key_code
    @key_code = key_code
  end
  
  # returns an array of :white and :black values, with :white(s) first
  def check pattern

    pattern = pattern.upcase

    if pattern == @key_code
      [:black, :black, :black, :black]
    else
      invalid_colors = pattern.tr(@key_code, '')
      pegs = [:white] * (pattern.length - invalid_colors.length)

      if @key_code[0] == pattern[0]
        pegs.shift
        pegs << :black
      end

      if @key_code[1] == pattern[1]
        pegs.shift
        pegs << :black
      end

      if @key_code[2] == pattern[2]
        pegs.shift
        pegs << :black
      end

      if @key_code[3] == pattern[3]
        pegs.shift
        pegs << :black
      end

      pegs      

    end
  end

end

```

That's all we need to do.

<pre class='console'>
$&gt; ruby  game_engine_test.rb -n /test_0[1-8]/
Loaded suite /tmp/release/mastermind/game_engine_test
Started
........

Finished in 0.000887716 seconds.

8 tests, 8 assertions, 0 failures, 0 errors, 0 pendings, 0 omissions, 0 notifications
100% passed

9011.89 tests/s, 9011.89 assertions/s
</pre>

The code expects that the key code contains upper letters but we do not ensure it in the `initialize` method (object creation). Later, we are going to improve the code.

### Test: only 4-character codes are valid

If the given pattern does not contain four colours the engine returns an empty array, we consider it as totally invalid.


* game_engine.rb

```ruby
class GameEngine

  def initialize key_code
    @key_code = key_code
  end
  
  # returns an array of :white and :black values, with :white(s) first
  def check pattern

    return [] unless pattern.length == 4

    pattern = pattern.upcase

    if pattern == @key_code
      [:black, :black, :black, :black]
    else
      invalid_colors = pattern.tr(@key_code, '')
      pegs = [:white] * (pattern.length - invalid_colors.length)

      if @key_code[0] == pattern[0]
        pegs.shift
        pegs << :black
      end

      if @key_code[1] == pattern[1]
        pegs.shift
        pegs << :black
      end

      if @key_code[2] == pattern[2]
        pegs.shift
        pegs << :black
      end

      if @key_code[3] == pattern[3]
        pegs.shift
        pegs << :black
      end

      pegs      

    end
  end

end

```

<pre class='console'>
$&gt; ruby  game_engine_test.rb -n /test_0[1-9]/
Loaded suite /tmp/release/mastermind/game_engine_test
Started
.........

Finished in 0.000871936 seconds.

9 tests, 9 assertions, 0 failures, 0 errors, 0 pendings, 0 omissions, 0 notifications
100% passed

10321.86 tests/s, 10321.86 assertions/s
</pre>

### Test: repeating same color produces correct feedback

While thinking about the `tr` method, we realized that it would do more than we expected... it is going to remove every occurrence of the colours, leading to a wrong number of correct colours. Say the key code is `WBGY` and the proposed pattern is `WWWW`, after the _translation_ we get an empty string which seems to mean that the key code contains 4 white colours but the given pattern contains one _white_ at the right position and three _whites_ wrongly ordered, strange isn't it?

Therefore, we added the 10th test to show the problem.

<pre class='console'>
$&gt; ruby  game_engine_test.rb -n /test_\\d\\d/
Loaded suite /tmp/release/mastermind/game_engine_test
Started
.........F
===============================================================================
Failure:
test_10_repeating_same_color_produces_correct_feedback(GameEngineTests)
/tmp/release/mastermind/game_engine_test.rb:48:in `test_10_repeating_same_color_produces_correct_feedback'
     45:   end
     46:   
     47:   def test_10_repeating_same_color_produces_correct_feedback
  =&gt; 48:     assert_equal [:black], @engine.check('GGGG')
     49:   end
     50:   
     51: end
&lt;[:black]&gt; expected but was
&lt;[:white, :white, :white, :black]&gt;

diff:
? [:white, :white, :white, :black]
===============================================================================


Finished in 0.005191698 seconds.

10 tests, 10 assertions, 1 failures, 0 errors, 0 pendings, 0 omissions, 0 notifications
90% passed

1926.15 tests/s, 1926.15 assertions/s
</pre>

We must write our own _translate_ method, which removes the right number of colours from the given pattern. We name the method `remove_correct_colors`, as it makes more sense.


* game_engine.rb

```ruby
  def remove_correct_colors pattern
    
    pattern_array = []
    
    pattern.each_char do |character|
      pattern_array << character
    end
    
    @key_code.length.times do |index|
      
      color = @key_code[index]
      
      position = pattern_array.index(color)
      
      if position
        pattern_array.delete_at(position)
      end
      
    end
    
    pattern_array.join
    
  end

```

Lines 3 to 7 convert the pattern to an array of characters named `pattern_array`.

For each character in the `key_code`, loop starting on line 9, we search if it appears in the `pattern_array` array (line 13), if so we remove the character (only once) from `pattern_array` (line 16).

After the loop (on line 21), `pattern_array` contains the characters (colours) that did not match and we pack them as a `String`.

We need to call the new method instead of `tr` to make the test pass.


* game_engine.rb

```ruby
class GameEngine

  def initialize key_code
    @key_code = key_code
  end
  
  # returns an array of :white and :black values, with :white(s) first
  def check pattern

    return [] unless pattern.length == 4

    pattern = pattern.upcase

    if pattern == @key_code
      [:black, :black, :black, :black]
    else

      invalid_colors = remove_correct_colors(pattern)

      pegs = [:white] * (pattern.length - invalid_colors.length)

      if @key_code[0] == pattern[0]
        pegs.shift
        pegs << :black
      end

      if @key_code[1] == pattern[1]
        pegs.shift
        pegs << :black
      end

      if @key_code[2] == pattern[2]
        pegs.shift
        pegs << :black
      end

      if @key_code[3] == pattern[3]
        pegs.shift
        pegs << :black
      end

      pegs      

    end
  end

  def remove_correct_colors pattern
    
    pattern_array = []
    
    pattern.each_char do |character|
      pattern_array << character
    end
    
    @key_code.length.times do |index|
      
      color = @key_code[index]
      
      position = pattern_array.index(color)
      
      if position
        pattern_array.delete_at(position)
      end
      
    end
    
    pattern_array.join
    
  end

end

```

<pre class='console'>
$&gt; ruby  game_engine_test.rb -n /test_\\d\\d/
Loaded suite /tmp/release/mastermind/game_engine_test
Started
..........

Finished in 0.00114602 seconds.

10 tests, 10 assertions, 0 failures, 0 errors, 0 pendings, 0 omissions, 0 notifications
100% passed

8725.85 tests/s, 8725.85 assertions/s
</pre>

### Test: engine is not case sensitive (2)

We said earlier that the code converts the pattern to upper letters so that it is not case sensitive but when we set the key code (the secret code) we don't do anything. We must store the key code as upper letters at creation time.


* game_engine.rb

```ruby
class GameEngine

  def initialize key_code
    @key_code = key_code.upcase
  end
  
  # returns an array of :white and :black values, with :white(s) first
  def check pattern

    return [] unless pattern.length == 4

    pattern = pattern.upcase

    if pattern == @key_code
      [:black, :black, :black, :black]
    else

      invalid_colors = remove_correct_colors(pattern)

      pegs = [:white] * (pattern.length - invalid_colors.length)

      if @key_code[0] == pattern[0]
        pegs.shift
        pegs << :black
      end

      if @key_code[1] == pattern[1]
        pegs.shift
        pegs << :black
      end

      if @key_code[2] == pattern[2]
        pegs.shift
        pegs << :black
      end

      if @key_code[3] == pattern[3]
        pegs.shift
        pegs << :black
      end

      pegs      

    end
  end

  def remove_correct_colors pattern
    
    pattern_array = []
    
    pattern.each_char do |character|
      pattern_array << character
    end
    
    @key_code.length.times do |index|
      
      color = @key_code[index]
      
      position = pattern_array.index(color)
      
      if position
        pattern_array.delete_at(position)
      end
      
    end
    
    pattern_array.join
    
  end

end

```

The last test, in the second test case, is about this case.

<pre class='console'>
$&gt; ruby  game_engine_test.rb
Loaded suite /tmp/release/mastermind/game_engine_test
Started
...........

Finished in 0.001119842 seconds.

11 tests, 11 assertions, 0 failures, 0 errors, 0 pendings, 0 omissions, 0 notifications
100% passed

9822.81 tests/s, 9822.81 assertions/s
</pre>



_Jean Lazarou  (Feb 28, 2016 - 08:23AM)_
