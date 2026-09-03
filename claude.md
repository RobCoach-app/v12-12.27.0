We have recently created a new option in the AI Task, called JSON Math Solver. In this option, the 'task instruction' input will take a maths equation, and the response will be a JSON object with the solution. The JSON object will have the following structure:
```json
{ 4
}
```

Instead of adding it as part of AI Task, I want to make it into it's own separate action or node. Can you do that for me?

It should have the same functionality - user adds a maths equation, and the action or node will return the JSON object with the solution, with just the number as the value.
eg. `2 + 2` will return `4`, the output of this action will be `{4}`.
