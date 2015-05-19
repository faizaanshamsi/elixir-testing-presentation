autoscale: true

# [fit] Testing in Elixir
# with ExUnit

---

# What We'll Cover:

## Test drive a Mix app

## Unit tests

## Test setup and callbacks

## Doc tests

---

![](../Images/excited-anchorman.gif)

# [fit] Let's write some tests

---

```
> mix new converter

> cd converter

> mix test

Compiled lib/converter.ex
Generated converter app
.

Finished in 0.02 seconds (0.02s on load, 0.00s on tests)
1 tests, 0 failures

Randomized with seed 419254
```

---

```elixir
# test/test_helper.exs
ExUnit.start()

# test/converter_test.exs
defmodule ConverterTest do
  use ExUnit.Case, async: true

  test "the truth" do
    assert 1 + 1 == 2
  end
end
```
---

# Let's add a few:

```elixir
# test/converter_test.exs

test "Fahrenheit to Celsius" do
  result = Converter.fahrenheit_to_celsius(32.0)
  assert result == {:ok, 0.0}
end

```
---

```elixir
# lib/converter.ex
def fahrenheit_to_celsius(num) do
  (num - 32.0) * (5.0 / 9.0)
end
```

---

```elixir
# test/converter_test.exs
test "Fahrenheit to Celsius" do
  result = Converter.fahrenheit_to_celsius(32.0)
  assert result == {:ok, 0.0}
end

test "Absolute Zero" do
  result = Converter.fahrenheit_to_celsius(-459.68)
  assert result == {:error, "Below Absolute Zero"}
end
```

---

```elixir
# lib/converter.ex
def fahrenheit_to_celsius(num) when num >= -459.67 do
  {:ok, (num - 32.0) * (5.0 / 9.0)}
end

def fahrenheit_to_celsius(_) do
  {:error, "Below Absolute Zero"}
end
```
---

```elixir
test "Fahrenheit to Celsius" do
  result = Converter.fahrenheit_to_celsius(32.0)
  assert result == {:ok, 0.0}
end

test "Absolute Zero" do
  result = Converter.fahrenheit_to_celsius(-459.68)
  assert  == {:error, "Below Absolute Zero"}
end

test "Precision" do
  {:ok, result} = Converter.fahrenheit_to_celsius(-458.683)
  assert_in_delta result, -272.6, 0.01
end
```

---

# [fit] ExUnit.Assertions

Message passing between processes:

* `assert_receive/3`
* `assert_received/2`

Dealing with exceptions:

* `assert_raise/2`
* `assert_raise/3`

---

# Additionally for each `assert` there's a matching `refute` clause

* `refute/1` & `refute/2`
* `refute_in_delta/4`
* `refute_receive/3` & `refute_received/2`

---

# How it all works

1. `ExUnit` is a Mix app, your `mix test` command starts it up.
2. `ExUnit.Case` imports callbacks, assertions, and doctests. Also provides the `test` macro we've been using.
3. `ExUnit.TestCase` has all of the `ExUnit.Test`
4. `ExUnit.Test` is a struct containing all the info about in individual test

---

# ExUnit.Callbacks

### `setup_all` runs once before first test in `ExUnit.TestCase` process

### `setup` runs before each test in same process as `ExUnit.Test`

### `on_exit` runs in seperate process from either the `ExUnit.TestCase` or `ExUnit.TestCase`, ensuring it completes even if the test process crashes

---

This will provide a message before and after the entire suite of tests in this file

```elixir
defmodule ConverterTest do
  use ExUnit.Case, async: true

  setup_all do
    IO.puts "Starting Test"

    on_exit fn ->
      IO.puts "Tests Complete"
    end

    :ok
  end
end
```
---
We can also provide context to *all* of our tests by passing a Dict in `setup_all`

```elixir
defmodule ConverterTest do
  use ExUnit.Case

  setup_all do
    IO.puts "Starting Test"

    on_exit fn ->
      IO.puts "Tests Complete"
    end

    {:ok, %{msg1: "Message 1", msg2: "Message 2"}}
  end

  test "Test 1", %{msg1: msg} do
    IO.puts msg
  end

  test "Test 2", %{msg2: msg} do
    IO.puts msg
  end
end
```
---

```
> mix test
Starting Test
Message 1
Message 2
.Tests Complete
.

Finished in 0.02 seconds (0.02s on load, 0.00s on tests)
2 tests, 0 failures

Randomized with seed 80369
```
---

# A more sophisticated example
![original](../Images/sophiscated_cat.png)

---

```elixir
defmodule ConverterTest do
  use ExUnit.Case

  setup_all do
    IO.puts "Starting Test"

    on_exit fn ->
      IO.puts "Tests Complete"
    end

    :ok
  end

  setup context do
    {:ok, %{quantity: context[:temp]}}
  end

  @moduletag temp: 32.0

  @tag temp: 55.0
  test "Test 1", %{quantity: temp} do
    IO.puts temp
  end

  test "Test 2", %{quantity: temp} do
    IO.puts temp
  end
end
```

---

```
> mix test
Starting Test
55.0
32.0
Tests Complete
.

Finished in 0.03 seconds (0.03s on load, 0.00s on tests)
2 tests, 0 failures

Randomized with seed 574641

```

---

# ExUnit.Doctest

Elixir treats documentation as a first class citizen, and this extends to Testing.

Commented code examples can become out of date if they're not updated in line with the code.

Elixir allows us to test our comments to keep them valid.

---

```elixir
defmodule Converter do
  @doc """
      iex> Converter.fahrenheit_to_celsius(32.0)
      {:ok, 1.0}
  """

  def fahrenheit_to_celsius(num) when num >= -459.67 do
    {:ok, (num - 32.0) * (5.0 / 9.0)}
  end

  def fahrenheit_to_celsius(_) do
    {:error, "Below Absolute Zero"}
  end
end

```

---

```
> mix test
Compiled lib/converter.ex
Generated converter app


  1) test doc at Converter.fahrenheit_to_celsius/1 (1) (ConverterTest)
     test/converter_test.exs:3
     Doctest failed
     code: Converter.fahrenheit_to_celsius(32.0) === {:ok, 1.0}
     lhs:  {:ok, 0.0}
     stacktrace:
       lib/converter.ex:8: Converter (module)



Finished in 0.04 seconds (0.04s on load, 0.00s on tests)
1 tests, 1 failures

Randomized with seed 155713
```

---
After updating the example to valid output:

```elixir
@doc """
    iex> Converter.fahrenheit_to_celsius(32.0)
    {:ok, 0.0}
"""
```

```
> mix test
Compiled lib/converter.ex
Generated converter app
.

Finished in 0.04 seconds (0.04s on load, 0.00s on tests)
1 tests, 0 failures

Randomized with seed 807347
```

---

# ExUnit.CaseTemplate

* Useful to DRY up your test suite
* Great if you have multiple test modules that need to share callbacks or functions
* An example of this I've come across in Phoenix is testing Serializer modules

---
```elixir
  # test/test_helper.exs
  defmodule ArithmeticCase do
    use ExUnit.CaseTemplate

    setup_all do
      IO.puts "Starting Arithmetic Module Test"

      on_exit fn ->
        IO.puts "Tests Complete"
      end

      {:ok, %{num: 1}}
    end
  end

  ExUnit.start()
```

---

```elixir
  defmodule AdditionTest do
    # In lieu of ExUnit.case
    use ArithmeticCase, async: true

    test "the truth", %{num: num} do
      assert num + num == 2
    end
  end

  defmodule SubtractionTest do
    use ArithmeticCase, async: true

    test "the truth", %{num: num} do
      assert num + num == 2
    end
  end
```
---

# Other cool stuff we didn't cover

* Filters - Allows you to pass command line arguments
* CaptureIO - Message passing to devices (for TDDing your swarm of drones)
* Structs - You can pass in Structs as the context of a test to DRY your test setup
* Everything I don't know && Everything I forgot

---

> When will he stop?

> I'd rather hear about Voorhees!

> I'm full of pizza and can't pay any more attention.

---

# Faizaan Shamsi

## ![inline](../Images/greenfield.png) Developer @ Greenfield
## ![inline](../Images/GitHub-Logos/GitHub_Logo.png) faizaanshamsi
## ![inline](../Images/Twitter_logo_blue.png) @faizaanshamsi
## ![inline](../Images/googlemail-128.png) faizaanshamsi@gmail.com
