---
sidebar_position: 4
---

# Input and output files

## Prompt Files

Prompt files are plain text files that contain the prompts you want to test. If you have only one file, you can include multiple prompts in the file, separated by the delimiter `---`. If you have multiple files, each prompt should be in a separate file.

You can use [Nunjucks](https://mozilla.github.io/nunjucks/) templating syntax to include variables in your prompts, which will be replaced with actual values from your test cases during evaluation.

Example of a single prompt file with multiple prompts (`prompts.txt`):

```
Translate the following text to French: "{{name}}: {{text}}"
---
Translate the following text to German: "{{name}}: {{text}}"
```

Example of multiple prompt files:

- `prompt1.txt`:

  ```
  Translate the following text to French: "{{name}}: {{text}}"
  ```

- `prompt2.txt`:

  ```
  Translate the following text to German: "{{name}}: {{text}}"
  ```

Prompts can be JSON too. Use this to configure multi-shot prompt formats:

```json
[
  {
    "role": "system",
    "content": "You are a translator can converts input to {{ language }}."
  },
  {
    "role": "user",
    "content": "{{ text }}"
  }
]
```

### Prompt functions


Prompt functions allow you to incorporate custom logic in your prompts. These functions are written in JavaScript or Python and are included in the prompt files with `.js` or `.py` extensions.

In the prompt function, you can access the test case variables through the `vars` object. The function should return a string or an object that represents the prompt.

#### Examples

A Javascript prompt function, `prompt.js`:

```javascript title=prompt.js
module.exports = async function ({ vars }) {
  return [
    {
      role: 'system',
      content: `You're an angry pirate. Be concise and stay in character.`,
    },
    {
      role: 'user',
      content: `Tell me about ${vars.topic}`,
    },
  ];
};
```

To reference a specific function in your prompt file, use the following syntax: `filename.js:functionName`:

```javascript title=prompt.js:prompt1
// highlight-start
module.exports.prompt1 = async function ({ vars }) {
// highlight-end
  return [
    {
      role: 'system',
      content: `You're an angry pirate. Be concise and stay in character.`,
    },
    {
      role: 'user',
      content: `Tell me about ${vars.topic}`,
    },
  ];
};
```

A Python prompt function, `prompt.py`:
```python title=prompt.py
import sys
import json

def generate_prompt(context):
    return f'Describe {context["vars"]["topic"]} concisely, comparing it to the Python programming language.'

if __name__ == '__main__':
    print(generate_prompt(json.loads(sys.argv[1])))
```

## Tests File

The tests file is an optional CSV file that can be used to define test cases separately from the main configuration.

The first row of the CSV file should contain the variable names, and each subsequent row should contain the corresponding values for each test case.

Vars are substituted by [Nunjucks](https://mozilla.github.io/nunjucks/) templating syntax into prompts. The first row is the variable names. All other rows are variable values.

Example of a tests file (`tests.csv`):

```
language,input
German,"Hello, world!"
Spanish,Where is the library?
```

The vars file optionally supports some special columns:

- `__expected`: A column that includes [test assertions](/docs/configuration/expected-outputs). This column lets you automatically mark output according to quality expectations.
- `__prefix`: This string is prepended to each prompt before it's sent to the API
- `__suffix`: This string is appended to each prompt before it's sent to the API

## Output File

The results of the evaluation are written to this file. Each record in the output file corresponds to a test case and includes the original prompt, the output generated by the LLM, and the values of the variables used in the test case.

For example outputs, see the [examples/](https://github.com/typpo/promptfoo/tree/main/examples/simple-cli) directory.

The output file is specified by the `outputPath` key in the promptfoo configuration.
