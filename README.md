# D&D NPC Generator (using an Variational Auto-Encoder)
Advait Ukidve - June 2023 - MSc. Creative Computing at UAL CCI

## Two Sentence Description    
This project uses a Variational Auto Encoder model and a custom built dataset (built on work listed further in the Readme) of human-created Dungeons and Dragons characters to create an NPC (Non-Playable Character) generator to be used by Dungeons and Dragons DMs in a pinch.  

## Important Links
- Demonstration video:
- Link to original Dataset:
- Custom created Dataset:
- Notebook containing model:
- Implementation:

# Project and Process Documentation

### The Idea

The idea for this project was born out of my own personal experience as a Dungeon Master (DM) playing D&D with a variety of groups and friends. Often, a DMs need to create characters (NPCs) on the fly that may be inconsequential to the game in the larger scope but need to be well thought in order to continue an immersive session. This happens frequently in a typical session of D&D as the group strays from the story the DM had previously written, as is the affordance of the game. In other cases, while writing a campaign, the DM might want to create a bunch of characters beforehand that fit the same role in the narrative. In both these scenarios, in order to create a truly compelling character, the DM turns to a personal list of previously-created NPC templates or, in most cases, a random NPC generator. The problem I always encountered with NPC generators that exist already is two-fold:
1. NPC Generators give seemingly completely random characters - generating their stats, both quantitative but most importantly qualitative, at random from templates provided in the [Dungeon Master's Guide](https://archive.org/details/dungeon-masters-guide/Dungeon%20Master%27s%20Guide/). The randomness can be a useful tool in some cases, but in some others, DMs prefer creating characters with a few context-specific requirements with a generator filling out the other more benign statistics to complete the character.
For eg: if I wanted to create characters quickly to fill out an alchemy shop's interiors in a little predominantly elven hamlet in my campaign's world, I already know I want an elf, with an affinity to magic in either a mercantile or a scholarly template. A machine is good at filling out the other 20 quantitative stats I need alongside it to make a deep character, but current systems do not allow one to predecide a few characteristics and fill out the rest driven by data.
2. These tools also give 'well-fleshed out' characters limiting the DM's scope for creativity. A perfect generator would instead give jumping-off points to create characters. The more there is available in a template, the less one explores outside it.

### Aim
The aims of the project were, hence, to create an NPC generator that:
1. Affords generation guided by pre-decided characteristic by mapping out the model's embedded space and movement required to get the kind of character one wants.
2. Provides a large number and variety of jumping-off points to build a character that the DM can then choose from according to their discretion.
3. Ensures the jumping off points are not completely random and work with each other cohesively. For example, having a '7 foot gnome mage wearing chainmail' makes very little sense but could also be compelling. A healthy mix of traits that work with each other like '3-5ft height for a gnome' and 'light armour/robes for a mage', with sometimes-absurd randomness mixed in would achieve this. This would be driven by a dataset 
4. Builds NPCs similar to Player Characters created by humans so they make for good storytelling at a granular level.


### Getting The Dataset
- In order to build a dataset that qualified Aim 3 listed above, I needed a large set of characters with traits that made sense, then edited to add a little randomness (perhaps in a 90-10 or 95-5 weight distribution of 'makes sense' vs. 'absurd but funny'. 
- I also needed a data set that already contained the quantitative measures of an NPC - traits that do not afford randomness per the rules of the game - based preferably on human-created characters.
- Fortunately, I found a dataset that qualified 60% of my requirements - [This one right here]

### Building on the Dataset

### How do the traits fit into an auto-encoder?
#### Race
#### Race

### Building the Model

### The Results

### Visualising the Embedded Space

### Limitations of the Project

### Evaluating the Project
