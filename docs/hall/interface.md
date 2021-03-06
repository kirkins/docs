---
navhome: /docs/
navuptwo: true
next: true
sort: 2
title: Hall interface
---

# Hall interface

This document describes the different interfaces Hall provides and the data that is accessible and modifiable through them. Knowledge of the Urbit application model (including [new gall](new-gall)) and [Hall's architecture](architecture) is assumed.

While the structures here are given in Hoon, they match fairly closely to their JSON equivalent. Most important to note is that `$%({$x y/z})` becomes accessible as `json.x.y`.


## Table of contents

* [Queries](#queries)
  * [/client](#client)
    * [/client prize](#client-prize)
    * [/client rumor](#client-rumor)
  * [/public](#public)
    * [/public prize](#public-prize)
    * [/public rumor](#public-rumor)
  * [/peers](#peers)
    * [/peers prize](#peers-prize)
    * [/peers rumor](#peers-rumor)
  * [/circle](#circle)
    * [/circle prize](#circle-prize)
    * [/circle rumor](#circle-rumor)
* [Actions](#actions)
  * [Circle configuration](#circle-configuration)
  * [Messaging](#messaging)
  * [Personal metadata](#personal-metadata)
  * [Changing shared UI](#changing-shared-ui)
  * [Miscellaneous changes](#miscellaneous-changes)


## Queries

Queries are the paths you pass into `%peer` moves. Internally, they get translated to a `++query` structure for easier handling. We'll be giving examples of valid query paths alongside the structures themselves.

### /client

To be able to keep certain UI elements like glyphs and local nicknames consistent across different Hall clients, they can query Hall for the current UI state.

```
++  query                                               ::
  $%  {$client $~}                                      :<  shared ui state
      ::  ....                                          ::
  ==                                                    ::
```

Valid paths include:

```
:>  all shared ui state
/client
```

#### /client prize

Contains a map of glyphs and the audiences that they map to, as well as a map of ships and their locally set nicknames.

```
++  prize                                               :>  query result
  $%  {$client prize-client}                            :<  /client
      ::  ...                                           ::
  ==                                                    ::
++  prize-client                                        :> shared ui state
  $:  gys/(jug char audience)                           :<  glyph bindings
      nis/(map ship nick)                               :<  local nicknames
  ==                                                    ::
```

#### /client rumor

Contains either a bound or unbound glyph and its target, or a ship with its new nickname. A nickname of `''` means the associated ship no longer has a nickname set for it.

```
++  rumor                                               :>  query result change
  $%  {$client rum/rumor-client}                        :<  /client
      ::  ...                                           ::
  ==                                                    ::
++  rumor-client                                        :<  changed ui state
  $%  {$glyph diff-glyph}                               :<  un/bound glyph
      {$nick diff-nick}                                 :<  changed nickname
  ==                                                    ::
++  diff-glyph  {bin/? gyf/char aud/audience}           :<  un/bound glyph
++  diff-nick   {who/ship nic/nick}                     :<  changed nickname
```


### /public

To aid in circle discoverability, users can add circles to their "public membership" list. This can then be queried for by, for example, a profile page.

```
++  query                                               ::
  $%  {$public $~}                                      :<  public memberships
      ::  ....                                          ::
  ==                                                    ::
```

Valid paths include:

```
:>  all public memberships
/public                                               
```

#### /public prize

Contains the set of circles the user has on their public list.

```
++  prize                                               :>  query result
  $%  {$public cis/(set circle)}                        :<  /public
      ::  ...                                           ::
  ==                                                    ::
```

#### /public rumor

Contains a circle that was either added or removed from the public list.

```
++  rumor                                               :>  query result change
  $%  {$public add/? cir/circle}                        :<  /public
      ::  ...                                           ::
  ==                                                    ::
```

### /peers

To allow a circle owner to inspect who is currently subscribed to their stories, they can issue a query to retrieve subscription data.

```
++  query                                               ::
  $%  {$peers nom/naem}                                 :<  readers of story
      ::  ....                                          ::
  ==                                                    ::
```

Query paths are structured as follows:

```
/peers/[circle-name]
```

Valid paths include:

```
:>  peers for circle %urbit-meta
/peers/urbit-meta
```

#### /peers prize

Contains a map of ships and the different queries they currently have active for the selected story.

```
++  prize                                               :>  query result
  $%  {$peers pes/(jar ship query)}                     :<  /peers
      ::  ...                                           ::
  ==                                                    ::
```

#### /peers rumor

Contains a ship and a query, and a flag to indicate whether that subscription has started or ended.

```
++  rumor                                               :>  query result change
  $%  {$peers add/? who/ship qer/query}                 :<  /peers
      ::  ...                                           ::
  ==                                                    ::
```

### /circle

Circle queries allow for the retrieving of data from stories. Their messages, configuration, and presences can all be accessed. Since this is a lot of data, there are lots of possibilities for filtering it built in to the query itself.


A quick refresher on the difference between "local" and "remote" presence and configuration: "local" means it pertains to the circle itself; "remote" means it pertains to one of its configured sources. The latter is primarily useful to clients when using a circle for aggregation, like the `%inbox`.

```
++  query                                               ::
  $%  $:  $circle                                       :>  story query
          nom/naem                                      :<  circle name
          wer/(unit circle)                             :<  from source
          wat/(set circle-data)                         :<  data to get
          ran/range                                     :<  query duration
      ==                                                ::
      ::  ....                                          ::
  ==                                                    ::
++  circle-data                                         :>  kinds of circle data
  $?  $grams                                            :<  messages
      $group-l                                          :<  local presence
      $group-r                                          :<  remote presences
      $config-l                                         :<  local config
      $config-r                                         :<  remote configs
  ==                                                    ::
++  range                                               :>  inclusive msg range
  %-  unit                                              :<  ~ means everything
  $:  hed/place                                         :<  start of range
      tal/(unit place)                                  :<  opt end of range
  ==                                                    ::
++  place                                               :>  range indicators
  $%  {$da @da}                                         :<  date
      {$ud @ud}                                         :<  message number
  ==                                                    ::
```

Query paths are structured as follows:

```
/circle/[circle-name]/(from-circle)/[what/data]/(range-start(/range-end))
(from-circle)  :  optional message source, ~ship/circle
[what/data]    :  one or more of grams, group, group-l, group-r,
               :  config, config-l, config-r, combined using /
(range)        :  an optional range with an optional end, its points denoted
               :  in either message number (@ud) or date (@da)
```

Valid paths include:

```
:>  get all messages from circle %urbit-meta
/circle/urbit-meta/grams
:>  get all messages, all presences and local configs from %urbit-meta
/circle/urbit-meta/grams/group/config-l
:>  get all messages %urbit-meta has heard from its source ~zod/fora
/circle/urbit-meta/~zod/fora/grams
:>  get the first 100 messages from %urbit-meta
/circle/urbit-meta/grams/0/99
:>  get the first 100 messages %urbit-meta has heard from its source ~zod/fora
/circle/urbit-meta/~zod/fora/grams/0/99
:>  get all messages from %urbit-meta, starting now
/circle/urbit-meta/grams/group/(scow %da now)
:>  get all messages from %urbit-meta, starting now, ending at the 100th message
/circle/urbit-meta/grams/(scow %da now)/99
:>  get local presences from %urbit-meta for a week
/circle/urbit-meta/group-l/(scow %da now)/(scow %da (add now ~d7))
```

#### /circle prize

Contains (where applicable) messages in envelopes (with message numbers), as well as local and remote configurations and presences.

```
++  prize                                               :>  query result
  $%  {$circle package}                                 :<  /circle
      ::  ...                                           ::
  ==                                                    ::
++  package                                             :>  story state
  $:  nes/(list envelope)                               :<  messages
      cos/lobby                                         :<  loc & rem configs
      pes/crowd                                         :<  loc & rem presences
  ==                                                    ::
```

#### /circle rumor

Contains a detailed change description of the data relevant to the query that changed.  

Messages are wrapped in envelopes to include their sequence number, and note the source they were heard from. Configuration and status changes specify the circle they apply to.

```
++  rumor                                               :>  query result change
  $%  {$circle rum/rumor-story}                         :<  /circle
      ::  ...                                           ::
  ==                                                    ::
++  rumor-story                                         :>  story rumor
  $%  {$new cof/config}                                 :<  new story
      {$gram src/circle nev/envelope}                   :<  new/changed message
      {$config cir/circle dif/diff-config}              :<  new/changed config
      {$status cir/circle who/ship dif/diff-status}     :<  new/changed status
      {$remove $~}                                      :<  removed story
  ==                                                    ::
++  diff-config                                         :>  config change
  $%  {$full cof/config}                                :<  set w/o side-effects
      {$source add/? src/source}                        :<  add/rem sources
      {$caption cap/cord}                               :<  changed description
      {$filter fit/filter}                              :<  changed filter
      {$secure sec/security}                            :<  changed security
      {$permit add/? sis/(set ship)}                    :<  add/rem to b/w-list
      {$remove $~}                                      :<  removed config
  ==                                                    ::
++  diff-status                                         :>  status change
  $%  {$full sat/status}                                :<  fully changed status
      {$presence pec/presence}                          :<  changed presence
      {$human dif/diff-human}                           :<  changed name
      {$remove $~}                                      :<  removed status
  ==                                                    ::
++  diff-human                                          :>  name change
  $%  {$full man/human}                                 :<  fully changed name
      {$handle han/(unit cord)}                         :<  changed handle
      {$true tru/(unit truename)}                       :<  changed true name
  ==                                                    ::
```


## Actions

Actions can be sent by poking Hall with data marked as `%hall-action`. Actions are used for all user operations. If an error or other unexpected behavior occurs while executing an action, Hall notifies the user by sending an `%app` message to their `%inbox`.

### Circle configuration

Since all of these apply to a specific circle, they all specify a name `nom` of the circle to operate on.

```
++  action                                              :>  user action
  $%  {$create nom/naem des/cord sec/security}          :<  create circle
      {$delete nom/naem why/(unit cord)}                :<  delete + announce
      {$depict nom/naem des/cord}                       :<  change description
      {$filter nom/naem fit/filter}                     :<  change message rules
      {$permit nom/naem inv/? sis/(set ship)}           :<  invite/banish
      {$source nom/naem sub/? srs/(set source)}         :<  un/sub to/from src
      ::  ...                                           ::
  ==
```

`%create`: Creates a circle with description `des` and security mode `sec`. If this mode is a whitelist, the user is automatically added to it.

`%delete`: Deletes the circle. If a reason `why` is provided, posts that as the last message to the circle before deleting it.

`%depict`: Set the description of the circle to `des`.

`%filter`: Set the filter (message sanitation rules) for the circle to `fit`.

`%permit`: Either invite or banish ships to/from the circle, modifying the access control list accordingly. Regardless of whether this actually makes any changes, sends an `%inv` message to the involved ships' `%inbox`es.

`%source`: Add or remove sources to/from the circle, un/subscribing it to/from the `grams`, `group-l` and `config-l` each one.

### Messaging

There are two interfaces for telling Hall to send a message. The first takes entire `thought`s, the second only `speech`es and the audience to send them to.

```
++  action                                              :>  user action
  $%  {$convey tos/(list thought)}                      :<  post exact
      {$phrase aud/audience ses/(list speech)}          :<  post easy
      ::  ...
  ==
```

`%convey`: Sends the thoughts to their audiences.

`%phrase`: Turns the speeches into thoughts by applying sane defaults for metadata (auto-generated `uid`, `now` for `wen`), and then sends those thoughts.

### Personal metadata

These concern the presence users have in circles they are participating in.

```
++  action                                              :>  user action
  $%  {$notify aud/audience pes/(unit presence)}        :<  our presence update
      {$naming aud/audience man/human}                  :<  our name update
      ::  ...                                           ::
  ==
```

`%notify`: Sets the user's presence in the audience circles to `pes`. A good client will automatically set these based on the user's activity. (`%talk` on typing, `%idle` on idle, `%gone` on sign-off.)

`%naming`: Sets the user's name (handle and real name) for the given audience. Good clients can display these in place of ship names.

### Changing shared UI

When the user makes any changes to shared UI elements (elements that should persist between clients), this has to be communicated to Hall.

```
++  action                                              :>  user action
  $%  {$glyph gyf/char aud/audience bin/?}              :<  un/bind a glyph
      {$nick who/ship nic/nick}                         :<  new identity
      ::  ...                                           ::
  ==
```


`%glyph`: Adds or removes a binding of a glyph to an audience.

`%nick`: Sets a local nickname for a ship. An empty nickname `''` means the ship has no nickname.

### Miscellaneous changes

```
++  action                                              :>  user action
  $%  {$public add/? cir/circle}                        :<  show/hide membership
      ::  ...                                           ::
  ==
```

`%public`: Adds or removes a circle to/from the user's public membership list.
