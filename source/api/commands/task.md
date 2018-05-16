---
title: task
comments: false
---

Execute code in node.js via the `task` plugin event.

{% note warning 'Anti-Pattern' %}
Don't try to start a web server from `cy.task()`.

Read about {% url 'best practices' best-practices#Web-Servers %} here.
{% endnote %}

# Syntax

```javascript
cy.task(event)
cy.task(event, arg)
cy.task(event, arg, options)
```

## Usage

**{% fa fa-check-circle green %} Correct Usage**

```javascript
// in test
cy.task('log', 'This will be output to the terminal')

// in plugins file
on('task', {
  log (message) {
    console.log(message)
    return null
  }
})
```

## Arguments

**{% fa fa-angle-right %} event** ***(String)***

An event name (can be any arbitrary string) that you handle via the `task` event in the {% url "`pluginsFile`" configuration#Folders-Files %}.

**{% fa fa-angle-right %} arg** ***(Object)***

An argument to send along with the event. Can be any type of value that can be serialized by [JSON.stringify()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify)  (string, boolean, array, object, etc.). Types such as functions, regular expressions, Symbols will be omitted or censored to `null`.

**{% fa fa-angle-right %} options** ***(Object)***

Pass in an options object to change the default behavior of `cy.task()`.

Option | Default | Description
--- | --- | ---
`log` | `true` | {% usage_options log %}
`timeout` | {% url `taskTimeout` configuration#Timeouts %} | {% usage_options timeout cy.task %}

## Yields {% helper_icon yields %}

`cy.task()` yields the value returned or resolved by the `task` event in the {% url "`pluginsFile`" configuration#Folders-Files %}.

# Examples

## Command

`cy.task()` provides an escape hatch for running arbitrary node code, so you can take actions necessary for your test outside the scope of Cypress. This is great for:

- Seeding your test database
- Storing state in node that you want persisted between tests
- Performing parallel tasks (like making multiple http requests outside of Cypress)
- Running an external process

In the `task` plugin event, the command will fail if `undefined` is returned. This helps catch typos or cases where the task event is not handled. If you don't need to return a value, explicitly return `null` to signal that the given event has been handled.

***Log a string to the terminal***

```javascript
// in test
cy.task('log', 'This will be output to the terminal')

// in plugins file
on('task', {
  log (message) {
    console.log(message) // This will log to the terminal
    return null // Signal that the 'log' event has been handled
  }
})
```

***Read a JSON file's contents***

Note: this serves as a demonstration only. We recommend using fixtures or {% url "`cy.readFile`" readfile %} for a more robust implementation of reading a file with Cypress.

```javascript
// in test
cy.task('readJson', 'cypress.json').then((data) => {
  // data equals:
  // {
  //   projectId: '12345',
  //   ...
  // }
})

// in plugins file
on('task', {
  readJson () {
    // reads the file relative to current working directory
    return fsExtra.readJson(path.join(process.cwd(), arg)
  }
})
```

## Options

***Change the timeout***

You can increase the time allowed to execute the task, although *we don't recommend executing tasks that take a long time to exit*.

Cypress will *not* continue running any other commands until `cy.task()` has finished, so a long-running command will drastically slow down your test cycle.

```javascript
// will fail if seeding the database takes longer than 20 seconds to finish
cy.task('seedDatabase', null, { timeout: 20000 });
```

# Notes

## Tasks Must End

***Tasks that do not end are not supported***

`cy.task()` does not support tasks that don't end, such as:

- Starting a server
- A task that runs a watch
- Any process that needs to be manually interrupted to stop

A task must end within the `taskTimeout` or Cypress will fail the current test.

# Rules

## Requirements {% helper_icon requirements %}

{% requirements task cy.task %}

## Assertions {% helper_icon assertions %}

{% assertions once cy.task %}

## Timeouts {% helper_icon timeout %}

{% timeouts task cy.task %}

# Command Log

***List the contents of cypress.json***

```javascript
cy.task('readJson', 'cypress.json')
```

The command above will display in the command log as:

![Command Log task](/img/api/task/task-read-cypress-json.png)

When clicking on the `task` command within the command log, the console outputs the following:

![console.log task](/img/api/task/console-shows-task-result.png)

# See also

- {% url `cy.exec()` exec %}
- {% url `cy.readFile()` readfile %}
- {% url `cy.request()` request %}
- {% url `cy.writeFile()` writefile %}
