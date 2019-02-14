![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Version](https://img.shields.io/badge/version-1.1.0-orange.svg)
![Godot Version](https://img.shields.io/badge/godot-3.1-brightgreen.svg)
![Status](https://img.shields.io/badge/status-beta-blue.svg)

## How to use
First of all, this solution is not an _addon_ yet, it's a pure gdscript _class_ i.e., doesn't extend any _Node_. I'll show it's work-flow.

### Constructor
```python
var parser = WhiskersParser.new(default_base_instance)
```

Returns an instance of the parser. The argument `default_base_instance` is an `Object` that will be used as context for expressions execution.

### Opening a Whiskers .json file
This class has a static function that opens an exported Whiskers file, it just opens the file, parse the JSON content with `parse_json()` function and return the data. The function is static because it should not have access to any variable or method from the instance.

```python
var dialogue_data = parser.open_whiskers(file_path)
```

### Starting a dialogue
```python
var block = parser.start_dialogue(dialogue_data, custom_base_instance)
```

The argument `custom_base_instance` sets a temporary base instance valid for the duration of this dialogue and it is optional considering you set a default base instance at the constructor.

Now it gets interesting. The `sart_dialogue` method returns a **block**, that is a `Dictionary` containing the content of a whiskers node, and every other node that it connects to, separated by category. An example **block** could be:

```python
{
	key = "Dialogue",
	text = "Do you like parties?",
	options = [
			{key = "Option", text = "Yes."},
			{key = "Option567", text = "No."}],
	expressions = [],
	dialogue = {}.
	condition = {},
	jump = {},
	is_final = false
}
```

Note that _options_ and _expressions_ are `Array` types, but `dialogue`, `condition` and `jump` are `Dictionary` types. That's because it is allowed only one of these per **block**.

### Navigating the dialogue
The method to proceed the dialogue is named `next` and it receives as argument the chosen option key. If there is no option then we call it without arguments. Either way, it will return the next **block** or an empty `Dictionary` if the dialogue ended.

The example **block** has two options, so to chose one we have to call:

```python
block = parser.next(block.options[0].key)
# For "Yes." or
block = parser.next(block.options[1].key)
# For "No."
```

I suggest creating `Button` nodes named by the options keys, and then using the names as argument. It is as simple as:

```python
for option in block.options:
	add_button(option)
# The add_button method has to be implemented by the user since it depends on the project
```

If you have a block like this:

```python
{
	key = "Dialogue",
	text = "I have something to tell you...",
	options = [],
	expressions = [],
	dialogue = {key = "Dialogue728"}.
	condition = {},
	jump = {},
	is_final = false
}
```

Calling `next` will create a **block** from the _Dialogue728_ node. So it allows a `Dialogue` node to be connected to another `Dialogue` node.

### Dealing with blocks
The blocks that are returned by `start_dialogue` and `next` are usually `Dialogue` types. That means you will just have to show the text and create the buttons if there are any. The `Expression`, `Condition` and `Jump` types are dealt with automatically by the **WhiskersParser**.

You can even use `String.format` method as described in the [Godot documentation](https://docs.godotengine.org/en/3.0/getting_started/scripting/gdscript/gdscript_format_string.html#format-method-examples) setting the `format_dictionary` variable like:

```python
parser.set_format_dictionary({"player_name" : "Godot-chan"})
```

And then any string **{player_name}** found in the dialogue texts will be replaced with **Godot-chan**. Note that I didn't say anything about **bbcode**, that's because it is a `RichTextLabel` method that's used out of this class.

## Considerations
I think it is really easy to use and the **block** abstraction worked well. I've already tested it with a bunch of dialogues. I didn't do anything about the `data.player_name` variable yet, but it is still there in `parser.data.player_name`. Also, any weird data or whiskers file and the class will print an **[ERROR]** or **[WARNING]** message on the terminal and return an empty `Dictionary`.
