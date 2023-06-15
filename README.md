# Fast Fiction
## D&D NPC Generator (using an Auto-Encoder)
<sub> Advait Ukidve - June 2023 - MSc. Creative Computing at UAL CCI </sub>

### Two Sentence Description    
This project uses an Auto Encoder model and a custom built dataset (built on work listed further in the Readme) of human-created Dungeons and Dragons characters to create an NPC (Non-Playable Character) generator to be used by Dungeons and Dragons DMs in a pinch. The generator allows for generation of charcters by using the embedded space of 3 Dimensional latent vectors in interesting ways.

#### Important Links
- Demonstration video:
- Notebook containing model:     


#### Other Links         
- Link to original Dataset:
- Custom created Dataset:
- Ground Truth Sheet:
- Normalised Dataset Used in Model:

# Project and Process Documentation

### The Idea

The idea for this project was born out of my own personal experience as a Dungeon Master (DM) playing D&D with a variety of groups and friends. Often, a DMs need to create characters (NPCs) on the fly. This happens frequently in a typical session of D&D as the group strays from the story the DM had previously written, as is the affordance of the game. In this case, in order to create a compelling character, the DM often turns to a random NPC generator. The problem I always encountered with NPC generators that exist already is two-fold:
1. NPC Generators give seemingly completely random characters - generating their stats, both quantitative but most importantly qualitative, at random from templates provided in the [Dungeon Master's Guide](https://archive.org/details/dungeon-masters-guide/Dungeon%20Master%27s%20Guide/). The randomness can be a useful tool in some cases, but in some others, DMs prefer creating characters with a few context-specific requirements with a generator filling out the other more benign statistics to complete the character.
For eg: if I wanted to create characters quickly to fill out an alchemy shop's interiors in a little predominantly elven hamlet in my campaign's world, I already know I want an elf, with an affinity to magic in either a mercantile or a scholarly template. A machine is good at filling out the other 20 quantitative stats I need alongside it to make a deep character, but current systems do not allow one to predecide a few characteristics and fill out the rest driven by data.
2. These tools also give 'well-fleshed out' characters limiting the DM's scope for creativity. A perfect generator would instead give jumping-off points to create characters. The more there is available in a template, the less one explores outside of it.

### Aim
The aims of the project were, hence, to create an NPC generator that:
1. Affords generation guided by pre-decided characteristic by mapping out the model's embedded space and movement required to get the kind of character one wants.
2. Provides a large number and variety of jumping-off points to build a character that the DM can then choose from according to their discretion.
3. Ensures the jumping off points are not completely random and work with each other cohesively. For example, having a '7 foot gnome mage wearing chainmail' makes very little sense but could also be compelling. A healthy mix of traits that work with each other like '3-5ft height for a gnome' and 'light armour/robes for a mage', with sometimes-absurd randomness mixed in would achieve this. This would be driven by a dataset 
4. Builds NPCs similar to Player Characters created by humans so they make for good storytelling at a granular level.


# Process

### Getting The Dataset
- In order to build a dataset that qualified Aim 3 listed above, I needed a large set of characters with traits that made sense, then edited to add a little randomness (perhaps in a 90-10 or 95-5 weight distribution of 'makes sense' vs. 'absurd but funny'. 
- I also needed a data set that already contained the quantitative measures of an NPC - traits that do not afford randomness per the rules of the game - based preferably on human-created characters.
- Fortunately, I found a dataset that qualified 60% of my requirements - [oganm's dnddata](https://github.com/oganm/dnddata)
- This dataset contained 10,894 characters and 16 traits each but were suffered from the inconsistencies that come from recording homebrew characters that don't fit fully within the official 5e structure of Dungeons and Dragons.

---

### Building on the Dataset

- I started from the [JSON file](./Dataset_Files/dnd_chars_all.json) from oganm and extracted the traits I wanted in my dataset as columns. 
- This file contained text that I could not directly use with an autoencoder. I decided to convert the text into integer based code first (as in [this file](./Dataset_Files/DnD_Dataset_Raw_Integers.xlsx)). 
- The integer-coding was done in accordance to a sheet called [GroundTruth](./Dataset_Files/GroundTruth.xlsx) to record the traits I wanted in my datasheet and how they would be represented in the final dataset.

<img src="./Images/1.png" width = 500px>   
<sub> Screenshot from GroundTruth to indicate the basis on which the integer coding was done </sub>  

- Then, I used the technique given on [this website](https://thatexcelsite.com/normalize-standardize-data-excel/) to standardise/normalise all the integers between 0.0 and 0.1 as floats given their respective ranges.
- Finally, I added columns to the file beyond what was present in the original dataset that would help my project further qualify Aim 2 given above. These included columns for NPC flaws, bonds, appearance traits, etc. that the D&D Handbook advices the DM to randomise from a list of 10-20. I believe this randomness benefits the character creation process but given a dataset that already contained these columns for each character, the outcome would be better (and not completely random).
- I recorded this in the sheet [(N)DnD_Dataset](./Dataset_Files/(N)DnD_Dataset.xlsx). This is the same file that is used in a CSV format in the final model training and a copy can hence be found in the [data](./data/dnd_dataset.csv) folder.   

<img src="./Images/1.png" width = 500px>   
<sub> Screenshot from final dataset as CSV showing normalised values used in the training to represent character traits </sub>  

---

### How do the traits fit into an auto-encoder?
Not all the traits fit into the normalised range CSV in a similar fashion. Instead, they use a few different techniques:  
#### Integer Encoding
Traits such as race, class, background, etc. have a finite number of possibilities each. All of these possibilities were given integer values ranging from 0 to 9 (at least) and 19 (at most). The ordering of these integers was based on some inherent meaning (for example, the races are ordered from smallest average size to largest, classes from least magical to most magical, etc.). These were then normalised as floats between 0.0 and 1.0.
These might have, in retrospect, been better off to perform one-hot encoding on. But given the time and the sheer scale, I chose integer encoding instead. This remains a limitation of the project - one that I aim to fix after the dust is settled.   

#### Multiples-of
Certain traits such as height, weight, and speed are in the format 'multiple-of'. These traits are tied intrinsicly to another one of the character's traits (for eg: height of a character can be given by the average height of the race the character belongs to). For these, I used float values between 0.5 and 1.5 as the trait. The decimal points can be multiplied with the average height of the character's race to calculate the character's height in inches.

#### One-hot Encoding
Some other features which are smaller in scope of possibilities such as the alignment of a character that can be given by two values - intention (good, evil, neutral) and method (chaotic, lawful, neutral) use one-hot encoding.

#### Tree Categorisation
Traits such as the kind of weapon a character carries of the armour they wear can have many possibilities that can be divided up into categories. For example, weapons have three levels of categorisation - magical/non-magical, martial/ranged, and finally the name of the weapon. Such traits use a 'tree categorisation' technique followed up by either integer encoding or one-hot encoding to denote the contents.

---

# The Model

I have documented the model training process in the same order as [the notebook](./Fast Fiction Notebook.ipynb) - with section numbers aligning with those in the notebook. This documentation explains key details and decisions while providing relevant screenshots without getting into any of the code I used. Please refer to the notebook for specific instances of the code.

### The Results

### Visualising the Embedded Space

## Evaluating
### Limitations of the Project

### Evaluating the Project
