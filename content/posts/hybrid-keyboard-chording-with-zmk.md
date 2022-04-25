---
title: "Hybrid Keyboard Chording With ZMK"
date: 2022-04-25T14:54:26+10:00
draft: false
---

Chording is the act of inputting text by pressing multiple keys at once, rather than typing sequentially as you normally would. There are chording approaches such as [Stenography](http://www.openstenoproject.org/) which court reporters often use in a similar way to shorthand to be able to input any word at the speed of speech, comfortably all day long.

This is impressive, but also takes years of practice to become proficient. The hybrid approach uses the regular sequential typing method, but with the ability to chord words using abbreviations that we define at any point in the sentence. The goal is not to be able to chord every word, but by focusing on the most common 200-300 words we can cover 65%+ of the English language.

[ZMK](https://zmk.dev/) is an open source firmware which can run on many bluetooth mechanical keyboards. It has many features, including [combos](https://zmk.dev/docs/features/combos) which we can use for chording, and [macros](https://zmk.dev/docs/behaviors/macros) which allow you to define the keys to be pressed when you press the combo.

You can see in action on my practicing chorded input video on [YouTube](https://youtu.be/WSWEC2Gn-5E)

## Creating Chords

I started by searching for a common word list, this page containing [The 500 most commonly used words](https://www.smart-words.org/500-most-commonly-used-english-words.html) seemed like a good starting point. I saved the list to a file:

`words.txt`

```
the
of
to
and
a
in
is
it
you
that
he
was
for
on
are
with
as
I
his
they
be
at
one
have
this
from
or
had
by
hot
but
some
what
there
we
can
out
other
were
all
your
when
up
use
word
how
said
an
each
she
which
do
their
time
if
will
way
about
many
then
them
would
write
like
so
these
her
long
make
thing
see
him
two
has
look
more
day
could
go
come
did
my
sound
no
most
number
who
over
know
water
than
call
first
people
may
down
side
been
now
find
any
new
work
part
take
get
place
made
live
where
after
back
little
only
round
man
year
came
show
every
good
me
give
our
under
name
very
through
just
form
much
great
think
say
help
low
line
before
turn
cause
same
mean
differ
move
right
boy
old
too
does
tell
sentence
set
three
want
air
well
also
play
small
end
put
home
read
hand
port
large
spell
add
even
land
here
must
big
high
such
follow
act
why
ask
men
change
went
light
kind
off
need
house
picture
try
us
again
animal
point
mother
world
near
build
self
earth
father
head
stand
own
page
should
country
found
answer
school
grow
study
still
learn
plant
cover
food
sun
four
thought
let
keep
eye
never
last
door
between
city
tree
cross
since
hard
start
might
story
saw
far
sea
draw
left
late
run
don't
while
press
close
night
real
life
few
stop
open
seem
together
next
white
children
begin
got
walk
example
ease
paper
often
always
music
those
both
mark
book
letter
until
mile
river
car
feet
care
second
group
carry
took
rain
eat
room
friend
began
idea
fish
mountain
north
once
base
hear
horse
cut
sure
watch
color
face
wood
main
enough
plain
girl
usual
young
ready
above
ever
red
list
though
feel
talk
bird
soon
body
dog
family
direct
pose
leave
song
measure
state
product
black
short
numeral
class
wind
question
happen
complete
ship
area
half
rock
order
fire
south
problem
piece
told
knew
pass
farm
top
whole
king
size
heard
best
hour
better
TRUE
during
hundred
am
remember
step
early
hold
west
ground
interest
reach
fast
five
sing
listen
six
table
travel
less
morning
ten
simple
several
vowel
toward
war
lay
against
pattern
slow
center
love
person
money
serve
appear
road
map
science
rule
govern
pull
cold
notice
voice
fall
power
town
fine
certain
fly
unit
lead
cry
dark
machine
note
wait
plan
figure
star
box
noun
field
rest
correct
able
pound
done
beauty
drive
stood
contain
front
teach
week
final
gave
green
oh
quick
develop
sleep
warm
free
minute
strong
special
mind
behind
clear
tail
produce
fact
street
inch
lot
nothing
course
stay
wheel
full
force
blue
object
decide
surface
deep
moon
island
foot
yet
busy
test
record
boat
common
gold
possible
plane
age
dry
wonder
laugh
thousand
ago
ran
check
game
shape
yes
hot
miss
brought
heat
snow
bed
bring
sit
perhaps
fill
east
weight
language
among
```

I wrote a nodejs script to generate unique chords for each word. I also decided to exclude words 2 characters or shorter in length so I could make it more worthwhile to chord compared to number of key you would need to press. The code isn't the cleanest, but it's really only a one use script anyway:

`filter.js`

```js
const events = require('events');
const fs = require('fs');
const readline = require('readline');

(async function processLineByLine() {
  try {
    const rl = readline.createInterface({
      input: fs.createReadStream('words.txt'),
      crlfDelay: Infinity
    });

    let used = {};
    rl.on('line', (line) => {
      let [word, keys] = line.split(' ');

      if (word.length < 3) {
        return;
      }
      let combo = "";
      let index = "";
      if (!keys) {
        keys = word;
        //first letters strategy
        const cs = keys.split('');
        
        for (let x = 0; x < cs.length; x++) {
          if (combo.split('').includes(cs[x])) {
            continue;
          }
          combo += cs[x];
          // need to sort combo to ensure no dupes with different order
          index = combo.split('').sort().join('');

          if (!used[index]) {
            break;
          }
        }


        // consonants strategy
        if (combo.length > 2) {
          let ccombo = "";
          let ccs = keys.replace(/[aeiou]/g, '').split('');
          //ensure first char is not stripped if vowel
          if (keys.charAt(0) !== ccs[0]) {
            ccs = [ keys.charAt(0), ...ccs];
          }

          for (let x = 0; x < ccs.length; x++) {
            // ignore letters occuring multiple letters
            if (ccombo.split('').includes(ccs[x])) {
              continue;
            }
            ccombo += ccs[x];
            // need to sort combo to ensure no dupes with different order
            let cindex = ccombo.split('').sort().join('');

            if (!used[cindex] && (cindex.length < index.length || used[index])) {
              index = cindex;
              combo = ccombo;
              break;
            }
          }
        }
      } else {
        index = keys.split('').sort().join('');
        combo = keys;
      }


      if (used[index]) {
        throw new Error(`Can't use combo '${combo}' for word '${word}' already used by ${used[index]}`)
      }
      used[index] = word;

      console.log(word, combo);
      });

      await events.once(rl, 'close');

  } catch (err) {
    console.error(err);
  }
})();
```

The approach that I took was to use the first letters of the word one by one until it found a combination that hasn't been used. This results in the most common 't' word 'the' becoming the combo 't', 'that' becoming 'th', and 'they' becoming 'the' etc.

If the combo is greater than 2 characters, another strategy is used which I called the 'consonants strategy'. This approach removes the vowels, then does a similar check on the first letters. Having multiple strategies results in more short combos.

Even with these 2 approaches, the script has a few collisions that it will error on which you will need to fix manually. If you edit `words.txt` and add the preferred combo after the word, the script will use that instead eg. `what wt` will set 'what' to 'wt'.

The final issue is that some combos just aren't physically possible to press on a normal keyboard layout. I just went through each word and attepted to press the combos and changed them if they didn't work. I use a [Colemak-DHm](https://colemakmods.github.io/mod-dh/) layout, so if you use Qwerty or another layout you will have to try to update these yourself. I have tested the first 300 words in this list.

I ran the script using the command:

```
node filter.js > chords.txt
```

`chords.txt`

```
the t
and a
you y
that th
was w
for f
are ar
with wi
his h
they the
one o
have ha
this thi
from fr
had hd
hot ho
but b
some s
what wt
there thr
can c
out ou
other otr
were we
all al
your yo
when wn
use u
word wd
how hw
said sa
each e
she sh
which whi
their thei
time ti
will wl
way wy
about ab
many m
then tn
them tm
would wo
write wr
like l
these ths
her he
long lg
make ma
thing tin
see se
him hi
two two
has has
look lo
more mo
day d
could co
come cm
did di
sound so
most ms
number n
who who
over ov
know k
water wte
than tha
call ca
first fi
people p
may my
down do
side si
been be
now no
find fn
any an
new ne
work wk
part pa
take ta
get g
place pl
made md
live li
where whe
after af
back bk
little lt
only onl
round r
man mn
year yr
came cam
show sho
every ev
good go
give gi
our or
under un
name nam
very v
through thro
just j
form fm
much mu
great gr
think tk
say sy
help hp
low low
line ln
before bf
turn tu
cause cs
same sam
mean me
differ df
move mv
right ri
boy bo
old old
too to
does ds
tell te
sentence snt
set st
three thre
want wnt
air ai
well wel
also aso
play pla
small sal
end ed
put pu
home hoe
read re
hand hn
port po
large lr
spell sp
add ad
even evn
land lan
here hr
must mus
big bi
high hg
such su
follow fw
act act
why why
ask ak
men men
change ch
went wen
light lgt
kind ki
off of
need nd
house hse
picture pi
try tr
again ag
animal ani
point pn
mother mtr
world wrl
near nr
build bu
self sl
earth ea
father ft
head hea
stand sta
own own
page pe
should shd
country cn
found fou
answer ans
school sch
grow gro
study stu
still sti
learn le
plant pln
cover cv
food fod
sun sn
four fur
thought tho
let let
keep ke
eye ey
never nv
last lst
door dr
between bw
city ci
tree tre
cross cr
since sin
hard har
start str
might mi
story sto
saw sw
far far
draw dra
left lf
late lat
run ru
don't don
while whie
press pr
close cl
night ni
real rel
life lif
few fe
stop stp
open ope
seem sem
together tg
next nx
white wht
children chi
begin bg
got got
walk wal
example ex
ease eas
paper pae
often oft
always ays
music msc
those thos
both boh
mark mr
book bok
letter ltr
until unt
mile mie
river rv
car car
feet fet
care care
second sec
group gru
carry cay
took tok
rain rai
eat eat
room rom
friend fri
began bgn
idea i
fish fh
mountain mnt
north nor
once onc
base bs
hear hear
horse hor
cut cu
sure sr
watch tch
color col
face fae
wood wod
main mai
enough eno
plain pli
girl gir
usual usa
young yn
ready rdy
above abo
ever evr
red red
list lis
though thg
feel fel
talk tak
bird br
soon son
body by
dog dg
family fam
direct dir
pose pos
leave lv
song sng
measure mea
state stae
product pro
black bl
short shr
numeral num
class cla
wind win
question q
happen hap
complete com
ship shi
area are
half hal
rock roc
order ord
fire fire
south sou
problem prb
piece pc
told tol
knew kn
pass pas
farm frm
top tp
whole whol
king kin
size sz
heard hrd
best bes
hour hour
better bet
true tru
during du
hundred hu
remember rem
step ste
early earl
hold hol
west wes
ground grn
interest intr
reach rch
fast fas
five fv
sing sing
listen list
six sx
table tab
travel tra
less les
morning mrn
ten ten
simple sim
several sv
vowel vw
toward towr
war war
lay ly
against agi
pattern pat
slow slo
center ce
love lov
person psn
money mon
serve ser
appear apr
road roa
map mp
science sci
rule rul
govern gv
pull pul
cold cld
notice ntc
voice voi
fall fal
power pw
town ton
fine fin
certain cer
fly fly
unit uni
lead ld
cry cry
dark drk
machine mch
note note
wait wai
plan plan
figure fg
star star
box bx
noun nou
field fie
correct crt
able abl
pound pou
done done
beauty bea
drive drv
stood std
contain cont
front fro
teach tc
week wek
final fnl
gave gve
green gre
quick qu
develop del
sleep slp
warm wrm
free fre
minute min
strong stro
special spe
mind mnd
behind bh
clear cle
tail tai
produce prd
fact fac
inch inc
nothing noth
course cou
stay sty
wheel whel
full fu
force frc
blue blu
object obj
decide dc
surface sur
deep dp
island isla
yet yt
busy bus
record rcd
boat boat
common cmn
gold gol
possible psb
plane plane
age age
wonder wnd
laugh lau
thousand thou
ago ago
ran ran
check che
game gm
shape shp
yes yes
brought bro
heat heat
snow snw
bed bed
bring bri
perhaps per
weight wg
language lng
among amo
```

## Configuring ZMK

Macros in ZMK have the following syntax:

```
macros {
  ZMK_MACRO(m_the,
      wait-ms = <0>;
      tap-ms = <10>;
      bindings = <&kp T &kp H &kp E &kp SPACE>;
  )
}
```

This types `t` `h` `e` `<space>`

To trigger it by a combo, you can use this syntax:

```
combos {
  compatible = "zmk,combos";
  combo_m_the {
    timeout-ms = <100>;
    key-positions = <31 32 13>;
    bindings = <&m_the>;
  };
}
```

The key positions are dependant on your keyboard, and start with 0 at the top left and increment left to right 1, 2, 3 etc. This is not easy to read so I created some definitions to help which you can add to the top of your keymap:

```
#define P_Q 0
#define P_W 1
#define P_F 2
#define P_P 3
#define P_B 4
#define P_J 5
#define P_L 6
#define P_U 7
#define P_Y 8
#define P_SEMI 9
#define P_A 10
#define P_R 11
#define P_S 12
#define P_T 13
#define P_G 14
#define P_M 15
#define P_N 16
#define P_E 17
#define P_I 18
#define P_O 19
#define P_Z 20
#define P_X 21
#define P_C 22
#define P_D 23
#define P_V 24
#define P_K 25
#define P_H 26
#define P_COMA 27
#define P_DOT 28
#define P_SQT 29
#define P_COMBO 31 32
```

The `P_COMBO` keys are what I have used to differentiate a normal keypress and a chorded one. I experimented with using only letter keys in the combo, but I found I kept triggering combos when I didn't intend to. It requires a very small combo timeout which is easy to miss also. My combo keys are left thumb key (space) and right thumb key (shift). You could also have a single dedicated combo key if you have the room.

I also created some variables for the timing settings:

```
#define COMBO_TIMEOUT 200
#define MACRO_TAP 10
#define MACRO_WAIT 0
```

There are a few other settings you will need to add to your *.conf file also to ensure you can press multiple keys at once and each key can have multiple combos:

```
CONFIG_ZMK_HID_REPORT_TYPE_NKRO=y
CONFIG_ZMK_COMBO_MAX_COMBOS_PER_KEY=512
CONFIG_ZMK_COMBO_MAX_KEYS_PER_COMBO=10
```

## Importing Chords Into Your Keymap

Managing your chords by editing the macros and combos manually is not very practical. The following script reads `chords.txt` and updates the keymap file. One challenge is knowing where to insert the macros/combos since the user may already have some manually created ones they want to keep. The script searches for a comment in the keymap to mark the start/end point to know where it is safe to update:

```
macros {
  // MACROS START
  // MACROS END
}
combos {
  compatible = "zmk,combos";
  // COMBOS START
  // COMBOS END
}
```

This is the script that performs the import:

`import-chords.js`

```js
const events = require('events');
const fs = require('fs');
const readline = require('readline');

function translateKeys(x) {
  switch(x) {
    case "'":
      return 'SQT';
      break;
    case '`':
      return 'BSPC';
      break;
    case '_':
      return 'SPC';
      break;
    case '.':
      return 'DOT';
      break;
    case '@':
      return 'AT';
      break;
    default:
      return x;
  }
}

function mapBindings(x) {
  if (x.match(/[A-Z]/)) {
    return `&sk LSHIFT &kp ${x.toUpperCase()}`
  }

  return `&kp ${translateKeys(x).toUpperCase()}`
}

(async function processLineByLine() {
  try {
    const keymap = process.argv[2];
    if (!keymap) {
      throw new Error(`Missing keymap filename, please pass as first argument`);
    }
    if (!fs.existsSync(keymap)) {
      throw new Error(`Unable to find keymap file ${keymap}`);
    }
    let rl = readline.createInterface({
      input: fs.createReadStream('chords.txt'),
      crlfDelay: Infinity
    });

    let macros = '';
    let combos = '';
    let used = {};

    rl.on('line', (line) => {
      let [word, keys] = line.split(' ');
      let index = keys.split('').sort().join('');
      if (used[index]) {
        throw new Error(`Can't use combo '${keys}' for word '${word}' already used by ${used[index]}`)
      }
      used[index] = word;
      const macro='m_' + word.split('').map(translateKeys).join('');
      const inputs = keys.toUpperCase().split('').map(translateKeys);
      const bindings = word.split('').map(mapBindings).join(' ') + ' &kp SPACE';
      macros += `    ZMK_MACRO(${macro},
        wait-ms = <MACRO_WAIT>;
        tap-ms = <MACRO_TAP>;
        bindings = <${bindings}>;
    )
`

      const positions = 'P_' + inputs.join(' P_');
    combos += `    combo_${macro} {
      timeout-ms = <COMBO_TIMEOUT>;
      key-positions = <P_COMBO ${positions}>;
      bindings = <&${macro}>;
    };
`
    });

    await events.once(rl, 'close');

    rl = readline.createInterface({
      input: fs.createReadStream(keymap),
      crlfDelay: Infinity
    });

    let output = '';
    let mode = 'normal';
    let foundMacros = false;
    let foundCombos = false;
    rl.on('line', (line) => {
      if (mode === 'normal') {
        output += line + '\n';
        if (line.includes('MACROS START')) {
          mode = 'macros';
        } else if (line.includes('COMBOS START')) {
          mode = 'combos';
        }
      } else if (mode === 'macros') {
        if (line.includes('MACROS END')) {
          foundMacros = true;
          output += macros + '\n' + line + '\n';
          mode = 'normal';
        }
      } else if (mode === 'combos') {
        if (line.includes('COMBOS END')) {
          foundCombos = true;
          output += combos + '\n' + line + '\n';
          mode = 'normal';
        }
      }
    });

    await events.once(rl, 'close');

    if (!foundMacros) {
      throw new Error(`Unable to find MACROS START/END, please add the comments to your keymap:
        macros {
          // MACROS START
          // MACROS END
        }
      `)
    }
    if (!foundCombos) {
      throw new Error(`Unable to find MACROS START/END, please add the comments to your keymap:
        combos {
          compatible = "zmk,combos";
          // COMBOS START
          // COMBOS END
        }
      `)
    }
    fs.writeFileSync(keymap, output, { encoding: "utf8", flag: "w", mode: 0o644 });
  } catch (err) {
    console.error(err);
  }
})();
```

You can run the script using the following command, replace cradio.keymap with your keymap filename:

```
node import-chords.js cradio.keymap
```

The keymap should be updated in the following way:

```
...
  macros {
    // MACROS START
    ZMK_MACRO(m_the,
        wait-ms = <MACRO_WAIT>;
        tap-ms = <MACRO_TAP>;
        bindings = <&kp T &kp H &kp E &kp SPACE>;
    )
    ZMK_MACRO(m_and,
        wait-ms = <MACRO_WAIT>;
        tap-ms = <MACRO_TAP>;
        bindings = <&kp A &kp N &kp D &kp SPACE>;
    )
    ZMK_MACRO(m_you,
        wait-ms = <MACRO_WAIT>;
        tap-ms = <MACRO_TAP>;
        bindings = <&kp Y &kp O &kp U &kp SPACE>;
    )
    ZMK_MACRO(m_that,
        wait-ms = <MACRO_WAIT>;
        tap-ms = <MACRO_TAP>;
        bindings = <&kp T &kp H &kp A &kp T &kp SPACE>;
    )
    ...
    // MACROS END
  };
  combos {
    compatible = "zmk,combos";
    // COMBOS START
    combo_m_the {
      timeout-ms = <COMBO_TIMEOUT>;
      key-positions = <P_COMBO P_T>;
      bindings = <&m_the>;
    };
    combo_m_and {
      timeout-ms = <COMBO_TIMEOUT>;
      key-positions = <P_COMBO P_A>;
      bindings = <&m_and>;
    };
    combo_m_you {
      timeout-ms = <COMBO_TIMEOUT>;
      key-positions = <P_COMBO P_Y>;
      bindings = <&m_you>;
    };
    combo_m_that {
      timeout-ms = <COMBO_TIMEOUT>;
      key-positions = <P_COMBO P_T P_H>;
      bindings = <&m_that>;
    };
    // COMBOS END
  };
```
Flash your keyboard as usual and you should be able to start chording!

I'm also experimenting with some other convenience combos which can help you with adding suffixes and typing emails etc. I am currently using these mappings to translate the word to a key but the downside is it stops you from entering an actual backtick or underscore 

- `backtick` Backspace
- `underscore` Space

```
`. .
`ing 'g
``ing 'v
`n't 'n
`s 's
`'s 'c
myemail@gmail.com ze
My_Name zn
```

You can see my keymap file [here](https://github.com/dlip/nixconfig/blob/master/keymaps/zmk/config/cradio.keymap)

## Learning Process

The amount of effort required may not be as high as Stenography, but it still requires some training to get the chords into your brain and muscle memory. My approach is to use the custom training mode in [monkeytype](https://monkeytype.com/) with 10 words at a time. I practice one group until I achieve at least 70 WPM with 100% accuracy before moving on to the next 10. I also sometimes check the random option to ensure I'm not just memorizing the words in order. Here is my training file:

`training.txt`

```
the and you that was for are with his they
one have this from had hot but some what there
can out other were all your when use word how
said each she which their time will way about many
then them would write like these her long make thing
see him two has look more day could come did
sound most number who over know water than call first
people may down side been now find any new work
part take get place made live where after back little
only round man year came show every good give our
under name very through just form much great think say
help low line before turn cause same mean differ move
right boy old too does tell sentence set three want
air well also play small end put home read hand
port large spell add even land here must big high
such follow act why ask men change went light kind
off need house picture try again animal point mother world
near build self earth father head stand own page should
country found answer school grow study still learn plant cover
food sun four thought let keep eye never last door
between city tree cross since hard start might story saw
far draw left late run don't while press close night
real life few stop open seem together next white children
begin got walk example ease paper often always music those
both mark book letter until mile river car feet care
second group carry took rain eat room friend began idea
fish mountain north once base hear horse cut sure watch
color face wood main enough plain girl usual young ready
above ever red list though feel talk bird soon body
dog family direct pose leave song measure state product black
short numeral class wind question happen complete ship area half
rock order fire south problem piece told knew pass farm
top whole king size heard best hour better TRUE during
hundred remember step early hold west ground interest reach fast
five sing listen six table travel less morning ten simple
several vowel toward war lay against pattern slow center love
person money serve appear road map science rule govern pull
cold notice voice fall power town fine certain fly unit
lead cry dark machine note wait plan figure star box
noun field correct able pound done beauty drive stood contain
front teach week final gave green quick develop sleep warm
free minute strong special mind behind clear tail produce fact
inch nothing course stay wheel full force blue object decide
surface deep island yet busy record boat common gold possible
plane age wonder laugh thousand ago ran check game shape
yes brought heat snow bed bring perhaps weight language among
```
