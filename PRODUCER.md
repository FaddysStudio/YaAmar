#!/usr/bin/env roll

# Music Producer

Work of Faddy Michel

In Solidarity with The People of Palestine till Their Whole Land is FREE

## Prerequisites

Since this file is a roll, it requires Faddy's Roll to be installed;
which in turn requires Node.js and NPM to be installed.

```sh
# After installing node and npm:
sudo npm i -g @faddys/roll
# Now, the roll command is available.
```

In addition, Csound must be installed for this roll to work.

## Usage

```sh
./PRODUCER.md NOTATION
```

## `examples/`

```roll
?# rm -fr examples ; mkdir examples
```

### `examples/scratch.no`

```roll
?# cat - > examples/scratch.no
```

```
+==

~ 105 4

o examples/

$ countdown 0/4 clap/1 1/4 clap/2 2/4 clap/3 3/4 clap/4

| $ maqsum 0/8 dom/01 1/8 tak/04 3/8 tak/07 4/8 dom/12 6/8 tak/06

| 0/4 sak/00 1/4 sak/08 2/4 sak/16 3/4 sak/23

-==
```

## Implementation

### `index.orc`

```roll
?# cat - > index.orc
```

```csound
//+==

sr = 48000
ksmps = 32
nchnls = 6
0dbfs = 1

instr 13, beat

iRate init 1 / abs ( p3 )
p3 *= 1000
SNote strget p4
strset p5, SNote

kLoop metro iRate

if kLoop == 1 then

schedulek 9 + frac ( p1 ), 0, 1, p5, p6, p7

endif

endin

instr 9, playback

SNote strget p4
p3 filelen SNote

aLeft, aRight diskin2 SNote

chnmix aLeft / ( p5 + 1 ), "left"
chnmix aRight / ( p6 + 1 ), "right"

endin

instr 4, record

SPath strget p4
SPath1 strcat SPath, "_1.wav"
SPath2 strcat SPath, "_2.wav"
SPath3 strcat SPath, "_3.wav"

aLeft1, aRight1 inch 1, 2
aLeft2, aRight2 inch 3, 4
aLeft3, aRight3 inch 5, 6

fout SPath1, -1, aLeft1, aRight1
fout SPath2, -1, aLeft2, aRight2
fout SPath3, -1, aLeft3, aRight3

endin

instr out

aLeft chnget "left"
aRight chnget "right"

aLeft clip aLeft, 0, 1
aRight clip aRight, 0, 1

outs aLeft, aRight

chnclear "left"
chnclear "right"

endin

schedule "out", 0, -1

//-==
```

### `@faddys/scenarist`

```roll
?# if [ ! -d node_modules/@faddys/scenarist ] ; then npm i @faddys/scenarist ; fi
```

### `index.mjs`

```roll
?# cat - > index.mjs
```

```js
//+==

import Scenarist from '@faddys/scenarist';
import { readFile } from 'node:fs/promises';

await Scenarist ( new class extends Array {

async $_producer ( $ ) {

try {

const notation = await readFile ( process .argv .slice ( 2 ) .shift () || 'examples/scratch.no', 'utf8' );

for ( let line of notation .trim () .split ( '\n' ) )
if ( ( line = line .trim () ) .length )
await $ ( ... line .split ( /\s+/ ) );

await $ ( '|' );

console .log ( this .map ( note => ( typeof note === 'object' ? this .note ( note ) : note ) ) .join ( '\n' ) );

} catch ( error ) {

console .error ( '#error', error ?.message || error );

}

}

[ '$~' ] ( $, tempo, bar, ... argv ) {

this .push ( `t 0 ${ this .tempo = tempo }`,
`v ${ this .bar = bar }` );

return $ ( ... argv );

}

time = 0;

[ '$|' ] ( $, ... argv ) {

return this .push ( `b ${ ++this .time * this .bar }` ), $ ( ... argv );

}

$$ ( $, label, ... argv ) {

this .label = label;
this [ label ] = [];
this [ '$' + label ] = ( $, ... argv ) => {

this .push ( ... this [ label ] );

return $ ( ... argv );

};

return $ ( ... argv );

}

left = 3;
right =3;

[ '$=' ] ( $, left, right, ... argv ) {

Object .assign ( this, { left, right } );

return $ ( ... argv );

}

#instance = 0
instance () { return ++this .#instance % 10 === 0 ? ++this .#instance : this .#instance }

$_director ( $, ... argv ) {

if ( ! argv .length )
return this .label;

const [ step, divisions ] = argv .shift () .split ( '/' );
const sound = argv .shift ();
const { time, left, right } = this;
const note = { step, divisions, sound, time, left, right };

this .push ( note );

if ( this .label ?.length )
this [ this .label ] .push ( note );

return $ ( ... argv );

}

note ( { step, divisions, sound, time, left, right } ) {

const instance = this .instance ();

return [

'i',
`13.${ instance }`,
`[${ step }/${ divisions }]`,
this .time,
`"equipment/${ sound }.wav"`,
instance,
left, right

] .join ( ' ' );

}

record = {}

 $o ( $, path ) {

this .push ( `i 4.${ this .record [ path ] = this .instance () } 0 -1 "${ path }"` );

}

} );

//-==
```

```roll
?# $ node index.mjs > index.sco
```

```roll
?# -1 csound -iadc -odac index.orc index.sco
```
