---
layout: post
title: example post 1
---

# Hello! This Is a Post

Prow scuttle parrel provost Sail ho shrouds spirits boom mizzenmast yardarm. Pinnace holystone mizzenmast quarter crow's nest nipperkin grog yardarm hempen halter furl. Swab barque interloper chantey doubloon starboard grog black jack gangway rutters.

Deadlights jack lad schooner scallywag dance the hempen jig carouser broadside cable strike colors. Bring a spring upon her cable holystone blow the man down spanker Shiver me timbers to go on account lookout wherry doubloon chase. Belay yo-ho-ho keelhaul squiffy black spot yardarm spyglass sheet transom heave to.

Trysail Sail ho Corsair red ensign hulk smartly boom jib rum gangway. Case shot Shiver me timbers gangplank crack Jennys tea cup ballast Blimey lee snow crow's nest rutters. Fluke jib scourge of the seven seas boatswain schooner gaff booty Jack Tar transom spirits.

# A Second Heading

Hardtack weigh anchor interloper lee aye crimp furl rum rope's end Yellow Jack. Loot gunwalls avast walk the plank dead men tell no tales pink prow heave down gangplank stern. Tack gun no prey, no pay landlubber or just lubber dead men tell no tales sheet keelhaul loot Admiral of the Black pirate. Fire in the hole overhaul spirits lass mizzen pinnace league gangplank log Chain Shot.

## Spoons and Forks

Lookout bring a spring upon her cable lugsail aye marooned fathom maroon draught matey ho. Spirits pillage chase gangplank measured fer yer chains main sheet tack red ensign gabion Sink me. Reef black jack lee lookout walk the plank flogging jib killick fathom Plate Fleet. Wherry topgallant lateen sail quarter transom long clothes spirits draft gangplank salmagundi.

### What Is a Spoon?

Scuttle no prey, no pay mizzen scuppers port stern come about boatswain transom cable. Transom interloper Buccaneer line grog blossom run a shot across the bow chase.

```haskell
import Html exposing (Html)
import Test exposing (..)
import List exposing (..)

main =
  Html.program
    { init = (initialModel, Cmd.none)
    , view = view
    , update = \msg model -> (model, Cmd.none)
    , subscriptions = \_ -> Sub.none
    }

-- MODEL

type alias Model = List Test

initialModel : Model
initialModel = tests

-- VIEW

view : Model -> Html a
view = viewTestResults

-- TESTS

tests : List Test
tests =
  [ test "1 + 1 is 2"
    (1 + 1 |> should (be 2))
  , test "1 + 1 is not 3"
    (1 + 1 |> shouldNot (be 3))
  , test "a passing test"
    ((1 + 1 |> should (be 2)) |> should (be Pass))
  , test "a failing test"
    ((1 + 1 |> should (be 3)) |> should (be (Fail "expected 2 to be 3")))
  , test "a passing test with negation"
    ((1 + 1 |> shouldNot (be 3)) |> should (be Pass))
  , test "a failing test with negation"
    ((1 + 1 |> shouldNot (be 2)) |> should (be (Fail "expected 2 not to be 2")))
  , test "filtering an empty list"
    (filter (always True) [] |> should (be []))
  , test "filtering with always true"
    (filter (always True) [1, 2, 3] |> should (be [1, 2, 3]))
  , test "filtering with always false"
    (filter (always False) [1, 2, 3] |> should (be []))
  ]
```

Hardtack weigh anchor interloper lee aye crimp furl rum rope's end Yellow Jack. Loot gunwalls avast walk the plank dead men tell no tales pink prow heave down gangplank stern. Tack gun no prey, no pay landlubber or just lubber dead men tell no tales sheet keelhaul loot Admiral of the Black pirate. Fire in the hole overhaul spirits lass mizzen pinnace league gangplank log Chain Shot.
