# mtg-analysis
A look at Magic: the Gathering cards using machine learning to analyze and predict words used in names, creature types, etc.


## Context
Magic: the Gathering is a trading card game where players use land cards (Plains, Island, Swamp, Mountain, Forest) to generate mana in one of five colors (white, blue, black, red, green) to cast spells in those colors. Each color has different mechanical characteristics and is associated with particular types of cards: most Elf cards are green, and almost all Angel cards are white.

Over its history, MTG has depicted many different worlds with unique creatures and events, often finding different ways to represent the same idea. With tens of thousands of uniquely named cards, I wanted to see what patterns had emerged over the years.


## Data Acquisition
I wanted every MTG card with its name, flavor text, color identity, card type, and creature type (if the card was a creature). Fortunately MTG players are obsessive by nature, and websites like the [MTGJSON](https://mtgjson.com) project have exhaustively catalogued every card and all its printings; there were also datasets available on Kaggle, though these were out of date so I didn't use them.

However, there were some problems:
1. The structure of the MTGJSON data file contained metadata and card data as the two highest fields, meanining the default parsing with pandas resulted in a dataframe with two columns, one of which contained a thirty-thousand-card-long JSON string.
2. The MTGJSON data files do not contain English flavor text. I searched thoroughly and compared various different types of JSON data files, but the only flavor text present was for non-English versions of cards.
3. MTGJSON stores a card's first printing information as the name of the set it was part of, not a particular date. Making the dataset easily readable would have required manually converting set codes into dates, which was beyond the scope of what I desired. (For mainline sets this wouldn't have been too much trouble, but the recent explosion in secondary products, alternate printings, and limited time FOMO sales has complicated matters.)

I was able to fix the first issue through trial and error, and assistance from my instructor, but the other problems would have resulted in scope creep.


## Methodology
The work is all done in main.ipynb, reading data from a JSON file in `/data` to create dataframes and then using pandas and string manipulation to make the data clean. The 100 most common words were isolated, then graphs are generated with matplotlib and saved manually to `/output`.

I had wanted to have a card's color (the mana required to play it) and its color identity (any secondary mana symbols that appear in its text box) to be considered as primary and secondary characteristics, but that was beyond my abilities at the time. Currently the data set uses a very quick and dirty solution: any card with a multicolored identity is copied across all of its colors - not an ideal solution, but it does allow for cards with differently-colored abilities to be counted.
Ex: a card that costs Green mana to play, but can produce Green or White or Blue mana would be counted as a green card, a blue card, and a white card.

The MTGJSON file is not included due to its size (over 100MB, most of which was non-English card information), but can be found [here](https://mtgjson.com/downloads/all-files/) in the AtomicCards link.

Ultimately I chose a simple word count approach, cleaning only stopwords and punctuation. If I had worked more efficiently then I think I would have been able to overcome these challenges and have more data, but my firsthand knowledge about MTG and its mechanics was enough to compensate.

## Analysis
Despite the above limitations, the output graphs were surprisingly insightful.

### The Basics
- White's most common words included typical holy things (`angel`, `guardian`, `light`, `spirit`, `soul`, `sun`) and defenders of holy things (`knight`, `paladin`, `hero`, `champion`, `guard`, `master`) with almost zero outliers. Funnily, `angel` is the single most common word, but if it were combined with `angelic` it would crush all other white namewords. The uncommon words are mostly variants that just aren't used as much (`cathar`, `enforcer`, `patrol`) and more common words (`law`, `ancient`, `scout`, `commander`).

- Blue's most common words are reptilian creatures (`drake`, `merfolk`, `dragon`, `serpent`), weather (`wind`, `storm`, `sea`, `mist`), and traditional wizards (`master`, `mage`, `mystic`, `sage`, `scholar`). However, plain words like `water`, `doctor`, `cloud`, `sorcerer`, and `herald` were rare.

- Black's most common namewords were evil-sounding things (`death`, `blood`, `dead`, `soul`, `hadow`, `vampire`, `witch`, `zombie`). However, the least common namewords contained mostly synonyms (`ghost`, `thief`, `evil`). Amusingly, `dark` was very common, whereas `darkness` was very rare.

- Red's most common words were dominated by ugly creatures (`goblin`, `dragon`, `hellkite`, `giant`, `elemental`, `ogre`), and various synonyms (`fury`, `rage`, `chaos`, `reckless`, `blood`, `lightning`, `flame`, `fire`). Red's least common namewords were orderly things (`disciple`, `veteran`, `rider`, `apprentice`) and plain words (`burn`, `wolf`, `fall`, `earth`, `crimson`).

- Green's most common namewords were `wurm`, `wild`, `growth`, `druid`, `hydra`, and surprisingly `spider`. `Elf` and `Elvish` were counted as separate words, but combined they would have been the single most common word. Words like `strength`, `giant`, `troll`, and `bear` were popular as well. Conversely, `scout`, `hero`, and `predator` were rare.


### Mascot Characters
- The Green mana charts reveal interesting details about mascot characters who have been used to sell the game for decades. A character like Garruk Wildspeaker (a big, tough, hairy wildman associated with green mana) is one of the least common namewords despite him appearing on a lot of cards and promotional material over the years.

### Misc
- I knew that `wurm` was a common green creature type and card nameword, but I was very surprised to see it at the top of the list (or near the top if the `elf`/`elvish` split were fixed). In recent years, the designers have decided that `hydra` sounds cooler, so that has become the standard "big, scary green creature" type.

- `Bola` is one of the least common black/red namewords, but this is not due to any number of spinning rocks connected by strings. The big bad evil dragon Nicol Bolas is associated with black and red mana, and I believe the stopword cleaning interpreted his name improperly (Bolas's Citadel -> `Bola` + `Citadel`).

- The `Sliver` creature type is among the most common in red, blue, and white, despite Slivers being present in all five colors (as a hive mind of creatures each strengthening one another). This indicates there are more multicolored Slivers that are partially red/blue/white than in other colors.

- Several Ravnican guild names (`Simic`, `Izzet`, `Gruul`) are among the least common namewords in their respective colors. This is counterintuitive as Ravnica has become probably the single most reused location in all of MTG fiction.

- Red cards are the only ones where plain word (`fire`/`flame`) is among the most common instead of a fantastical synonym.

- I was surprised to see `Barbarian` and `Crimson` among the rarest red namewords, but `Barbarian` may have placed higher if I had included creature types.

- Red had the biggest disparity between the top of the most common namewords and the secondary ones, with `Goblin` having more than double the count of `Dragon`. Even combining `Dragon` and `Hellkite` wouldn't have closed the gap.

- `Guildmage` was one of red's more common rare words, which is odd because `guildmage` cards are very specifically named and evenly distributed across every color. Thus there must be other names competing for space at the bottom in other colors.


## Conclusions and Future Versions
On a basic level, a pass of NLP should be applied to combine synonyms like `elf` and `elvish`. However, an MTG card is not just its name and characteristics, but also its art and setting. Cards like Shock (a simple red card that deals damage) have been reprinted with various new illustrations over the years, depicting many characters across many worlds. Therefore, even though a card may not contain the name Garruk Wildspeaker, it could certainly depict him in the art or mention him in the flavor text.

In order to get a better notion of how colors have been associated with certain themes (especially characters/lore that is original to MTG), it would be necessary to analyze the names and colors of each card, the characters depicted in the art, and have real time data for a card's first printing.

But why? Frankly, MTG has produced vanishingly little original worldbuilding or storytelling in recent years, and there have been a lot of "return to former world" sets to make it easier to keep up with the absurd number of products and cards printed. This is the type of thing that is obvious to anyone who has played the game before about 2018-2019, but is hard to quantify.

The game's sensibilities have also changed in no small ways: while a big dude like Garruk Wildspeaker and his hordes of wild animal friends were considered a cool way to sell the game to young boys in the 2000s, modern tastes consider such demographics unpalatable and the mascots have changed accordingly.