# pi-stepper
## A Stepper Motor Control Module

_by Mike Green_

## Introduction

`pi-stepper` is a flexible control class for stepper motors, forked and updated from my earlier module [wpi-stepper](https://github.com/mikedamage/wpi-stepper).

In place of WiringPi, which hasn't seen an update to its Node bindings in a couple of years, I'm using [onoff](https://www.npmjs.com/package/onoff). I also removed Babel and upped the minimum Node version required to use this module to 10.

`pi-stepper` allows you to wire your motor and controller however you prefer, and you can also program your own pin activation sequences by simply feeding some arrays of 1's and 0's to the `Stepper` class. This could be useful not only for driving stepper motors, but also for controlling anything that requires a repeating sequence of activation and deactivation.

## Installation

```sh
npm install --save pi-stepper
```

## Usage

```js
const { Stepper } = require('pi-stepper');
```

### Example

```js
const pins = [
  17, // A+
  16, // A-
  13, // B+
  12  // B-
];
const motor = new Stepper({ pins, steps: 200 });

motor.speed = 20; // 20 RPM

// Move the motor forward 800 steps (4 rotations), logging to console when done:
motor.move(800).then(() => console.log('motion complete'));
```

Additionally, `Stepper` is an `EventEmitter`, so you can subscribe to various events emitted by the class throughout its life cycle:

```js
motor.on('start', () => console.log('Starting to move!'));
motor.on('cancel', () => console.log('Stopping that. Doing this instead!'));
motor.on('complete', () => console.log('I\'m all finished!'));

motor.move(800);
motor.move(-800);

// => "Starting to move!"
// => "Stopping that. Doing this instead!"
// => "Starting to move!"
// (a few seconds later...)
// => "I'm all finished!"

```

### Custom Activation Modes

`pi-stepper` comes configured for a 4-wire stepper motor out of the box, and thus far that's all I've tested it with. However, you can easily use the `Stepper` class to drive other types of motors with different numbers of wires by passing it a custom `mode` option when you initialize an instance.

Activation modes are arrays of arrays, whose inner members are either `1` or `0` and correspond to each pin, in the order you first specified them. `pi-stepper` exports two available activation modes out of the box, and they look like this:

#### `DUAL` _(this is the default mode)_

Use this activation mode if you're driving a bipolar, 4-wire stepper motor:

```js
const { MODES, Stepper } = require('pi-stepper');

const pins = [ 17, 16, 13, 12 ];
const mode = MODES.DUAL;
/*
[
  [ 1, 0, 0, 1 ], // Pin states: (17: on, 16: off, 13: off, 12: on)
  [ 0, 1, 0, 1 ],
  [ 0, 1, 1, 0 ],
  [ 1, 0, 1, 0 ]
]
*/

const motor = new Stepper({ pins, mode });
```

#### `SINGLE` _(for unipolar motors)_

```js
const { MODES, Stepper } = require('pi-stepper');

const pins = [ 17, 16, 13, 12 ];
const mode = MODES.SINGLE;
/*
[
  [ 1, 0, 0, 0 ], // Pin states: (17: on, 16: off, 13: off, 12: off)
  [ 0, 1, 0, 0 ],
  [ 0, 0, 1, 0 ],
  [ 0, 0, 0, 1 ]
]
*/

const motor = new Stepper({ pins, mode });
```

As the motor turns, the `Stepper` class will step through these activation modes and apply the appropriate states to the pins you register. For both of the included modes, the pattern repeats every 4 steps. This can and should vary depending on how many wires your motor has.

#### Half-Stepping

_TODO_

## Full API

See the [API documentation](doc/api.md).

## License

`pi-stepper` is released under the terms of the [MIT License](./LICENSE).
