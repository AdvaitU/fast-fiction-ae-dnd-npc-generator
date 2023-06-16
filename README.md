# Fast Fiction
## D&D NPC Generator (using an Auto-Encoder)
<sub> Advait Ukidve - June 2023 - MSc. Creative Computing at UAL CCI </sub>

### Two Sentence Description    
This project uses an Auto Encoder model and a custom built dataset (built on work listed further in the Readme) of human-created Dungeons and Dragons characters to create an NPC (Non-Playable Character) generator to be used by Dungeons and Dragons DMs in a pinch. The generator allows for generation of charcters by using the embedded space of 3 Dimensional latent vectors in interesting ways.

#### Important Links
- [Demonstration video](https://youtu.be/l0tCYqb48eA)
- [Notebook containing model](./FasFiction-Notebook.ipynb)    

#### This Repo Contains:      
- [Notebook containing model](./FasFiction-Notebook.ipynb)   
- [Dataset used for training](./data/dnd_dataset.csv)
- Models saved as JSONs
- Weights saved as H5s
- Legible [Excel Sheets](./Dataset_Files/) referenced in the Process Documentation
- [Images used in this Readme](./Images/)

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

<img src="./Images/1.png" width = 500px align = center><sub> Screenshot from GroundTruth to indicate the basis on which the integer coding was done </sub>     


- Then, I used the technique given on [this website](https://thatexcelsite.com/normalize-standardize-data-excel/) to standardise/normalise all the integers between 0.0 and 0.1 as floats given their respective ranges.
- Finally, I added columns to the file beyond what was present in the original dataset that would help my project further qualify Aim 2 given above. These included columns for NPC flaws, bonds, appearance traits, etc. that the D&D Handbook advices the DM to randomise from a list of 10-20. I believe this randomness benefits the character creation process but given a dataset that already contained these columns for each character, the outcome would be better (and not completely random).
- I recorded this in the sheet [(N)DnD_Dataset](./Dataset_Files/(N)DnD_Dataset.xlsx). This is the same file that is used in a CSV format in the final model training and a copy can hence be found in the [data](./data/dnd_dataset.csv) folder.   

<img src="./Images/2.png" width = 1000px>   
<sub> Screenshot from final dataset as CSV showing normalised values used in the training to represent character traits </sub>     


---

### How do the traits fit into an auto-encoder?
Not all the traits fit into the normalised range CSV in a similar fashion. Instead, they use a few different techniques:  
#### Integer Encoding
Traits such as race, class, background, etc. have a finite number of possibilities each. All of these possibilities were given integer values ranging from 0 to 9 (at least) and 19 (at most). The ordering of these integers was based on some inherent meaning (for example, the races are ordered from smallest average size to largest, classes from least magical to most magical, etc.). These were then normalised as floats between 0.0 and 1.0.
These might have, in retrospect, been better off to perform one-hot encoding on. But given the time and the sheer scale, I chose integer encoding instead. This remains a limitation of the project - one that I aim to fix after the dust is settled.   

#### Multiples-of
Certain traits such as height, weight, and speed are in the format 'multiple-of'. These traits are tied intrinsically to another one of the character's traits (for eg: height of a character can be given by the average height of the race the character belongs to). For these, I used float values between 0.5 and 1.5 as the trait. The decimal points can be multiplied with the average height of the character's race to calculate the character's height in inches.

#### One-hot Encoding
Some other features which are smaller in scope of possibilities such as the alignment of a character that can be given by two values - intention (good, evil, neutral) and method (chaotic, lawful, neutral) use one-hot encoding.

#### Tree Categorisation
Traits such as the kind of weapon a character carries of the armour they wear can have many possibilities that can be divided up into categories. For example, weapons have three levels of categorisation - magical/non-magical, martial/ranged, and finally the name of the weapon. Such traits use a 'tree categorisation' technique followed up by either integer encoding or one-hot encoding to denote the contents.

---

# The Model

I have documented the model training process in the same order as [the notebook](./FastFiction-Notebook.ipynb) - with section numbers aligning with those in the notebook. This documentation explains key details and decisions while providing relevant screenshots without getting into any of the code I used. Please refer to the notebook for specific instances of the code.

### 1. The Data
- First the data is imported as a pandas dataframe called 'df' and split into x_train and x_test dataset in a 80-20 ratio. (Thanks to the [technique](https://stackoverflow.com/questions/24147278/how-do-i-create-test-and-train-samples-from-one-dataframe-with-pandas) I found on StackOverflow).
- The CSV dataset contains 10,894 rows and 31 columns.
- x_train hence has the shape (8715,31) and x_test has the shape (2719,31).
- No y_train and y_test were needed as the model is an autoencoder with the target being the same as the original dataset.

### 2. The Model 

- I chose to use an auto-encoder for the purpose. The auto-encoder is built using a separate encoder and decoder trained together.
- The model was based on the [VAE Notebook](https://git.arts.ac.uk/rfiebrink/ExploringMachineIntelligence_Spring2023/blob/main/week4/ConvolutionalVAE.ipynb) we explored in class in Week 4
- Both the Encoder and Decoder 9-10 connected layers with the majority being fully connected dense layers and an Input, BatchNormalisation and RELU Activation layer each.
- The encoder takes the shape (n,31) as an input and passes it through increasingly smaller layers 3x(n,31), 2x(n,15), 2x(n,7), 1x(n,3). The decoder works in the exact opposite fashion.
- The latent vector is of the shape (n,3) having 3 dimensions which make it easy to visualise the embedded space. I also tried reducing the dimensionality to 1, 2, and 4 but had similar results in terms of loss.

<img src="./Images/3.png" width = 1000px>   
<sub> Diagram of the Autoencoder model (Representational) </sub>  

- The model uses the optimiser 'adam'. I tried using multiple optimisers but 'adam' proved to be the most reliable.
- It uses 'mse' loss function to calculate loss as the target data is not all categorical or binary.
- I trained the model with a batch size of 32 for 500 epochs.

## 3. Evaluating the Model

- The model performed pretty well during the training with the loss consistently below 0.1.

<img src="./Images/4.png" width = 500px align = center>   
<sub> Graph showing loss and val_loss over 500 epochs </sub>  

- In order to evaluate the model further, I ran the test data through the aut-encoder and plotted the original and reconstructed vectors in a line graph.
- The reconstructed vectors weren't very accurate but followed the general pattern of the original vectors.   

<img src="./Images/5.png" width = 500px>   <img src="./Images/6.png" width = 500px>   
<sub> Line Diagrams showing the original vectors and reconstructed vectors from (left) the training set and (right) the testing set </sub>  

- Next, I plotted the embedded space by creating a 3D scatter diagram of the latent vectors created by encoding the training set. (Thanks to [this tutorial](https://www.geeksforgeeks.org/3d-scatter-plotting-in-python-using-matplotlib/ for teaching me how to do 3D scatter Plots with Matplotlib))
- Here, I first encountered the problem with this project I explain in detail at the end - that every time the model trained, it trained differently - often losing accuracy in reconstructing specific columns of data.
- Here is an array of interesting looking but indicative 3D scatters I got after training the model multiple times:   

<img src="./Images/7.png" width = 500px>   <img src="./Images/8.png" width = 500px>    <img src="./Images/9.png" width = 500px>   <img src="./Images/10.png" width = 500px>   <img src="./Images/11.png" width = 500px>   <img src="./Images/12.png" width = 500px>   
<sub> Visualisations of the embedded space as a 3D scatter plot </sub>  

## 4. Methods to Generate New Characters
- Next, I defined functions to generate new characters using the reconstructed vectors.
- The method uses individual values from the reconstructed vector's numpy array and runs them through functions that round, multiply, and/or categorise the resultant numbers and output text strings.
- Further functions use these strings and arrange/format them to (a) Create a detailed character description for the DM, (b) Create a Stats Sheet for the generated character for easy reference, and (c) Create prompts for portrait generation using Stable Diffusion.
- These functions are relatively straightforward and can be found in section 4.2 of the notebook.

---

## 5. Generating Characters in Meaningful Ways

#### Generating Random characters in the embedded space
- First, we create latent vectors of shape (1,3) and randomise their elements within the scope of the embedded space by calculating the min and max values found in the latent vectors in the previously visualised latent space.
- These are then decoded and sent through the functions created in Step 4 to create characters.
- This method simply illustrates the project's ability to create characters that are similar to but not the same as characters from the training set.   
- I also passed the generated descriptions through a stable diffusion model within the same notebook as an experiment but that did not yield the results I was looking for. Instead, I then passed them through Dall-E 2. The portraits displayed here are generated in Dall-E 2.

<img src="./Images/13.png" width = 320px>   <img src="./Images/14.png" width = 320px>   <img src="./Images/15.png" width = 320px>   <img src="./Images/16.png" width = 500px>   
<img src="./Images/17.png" width = 500px>   <img src="./Images/18.png" width = 500px>   
<sub> Three examples of random characters generated using the AE Model </sub>  

#### 5.2 Generating Characters in Between Randomly Generated Characters
- Next we use the midpoint formula (i.e. (x1+x20/2, (y1+y2)/2, (z1+z2)/2) to find the co-ordinates of the midpoint between two of the previously generated random characters.
- These midpoints are passed through the decoder to create new and unique characters.
- The three generated descriptions and portraits are two of the characters from the previously generated lineup (left and right) and their newly generated midpoint (centre).

<img src="./Images/19.png" width = 320px>   <img src="./Images/20.png" width = 320px>   <img src="./Images/21.png" width = 320px>   <img src="./Images/22.png" width = 1000px>   
<sub> Generating 'midpoint' characters </sub>  

#### 5.2 Generating Similar Characters
- Next, we generate characters in a similar mould as previously generated characters. To do this, we can look at the latent vector of a character we like but are not entirely happy with.
- Then, we create another latent vector by adding random variation to its x,y, and z co-ordinates within a pre-defined range.
- The bigger the range and the more disproportional it is (between x, y, and z variance), the further away the generated character will be from the original character.
- Based on my tinkering, a range of +-0.15 from the original latent vector values gives characters that are mostly similar, which is what I was looking for. Generating using the same formula over and over again brought me characters that are similar but different enough.

<img src="./Images/23.png" width = 1000px>
<sub> (in order) The two latent vectors; lat3; lat3_likeness </sub>  

#### 5.3 Generating characters with specific traits
- Finally, we generate characters with predecided traits as was one of the main aims of the project.
- After playing around with the latest generated embedded space, I could verifieably say that the characters with latent vectors x-values around and y-values around generated characters of the race tiefling.
- Similarly, latent vectors with z-values around generated Lawful Good Characters.
- Creating latent vectors with these x,y,z values respectively would then, in theory, generate tiefling/chaotic good characters.

<img src="./Images/24.png" width = 600px>   <img src="./Images/25.png" width = 400px>
<sub> Characters with specific tiefling and chaotic good traits </sub>  

- Thye obvious limitation of this technique was the change in the embedded space after every training instance. Knowing what values generate what is incidental to that training cycle and I believe fixing the problems inherent with the training data could, in the future, let me create the same embedded space and study it at scale to give definitive clusters as opposed to the above incidental variants. But I believe the technique is there and so is the opportunity.

## 5.4 Evaluating the Project

### Limitations of the Project
- The major limitation of the project in its current form is how it is training creates a different embedded space every time it is run. 
- The techniques to extract the necessary information in multiple ways and varying aims work well but are contingent on the embedded space being a useful model. This limitation exists because of the way the dataset is constructed, ultimately, with some attributes missing obvious patterns while other being a result of accurate data being unavailable. 
- The hypothesis is that with a well constructed dataset, the model would relibly train to create similar embedded spaces that the techniques in 4. and 5. can be used on to deliver the intended aim of the project.   
- Another major limitation for the project was in how the data set was built. One-hot encoding could have proved to be more useful than the integer based encoding I tried while constructing large scope traits. 
- Additionally, the lack of necessary data for some of the traits I desired the model to generate meant I had to resort to generating these traits for the existing characters at random. While randomness does allow for some novel creations, it does not truly support the spirit of this project and I would hope to create a dataset that also captures these additional features in order to build a better model in the future.

### Reflection
- Ignoring the limitations, my hypothesis that I could use an auto-encoder to generate numerical (and hence textual) data to create unique D&D characters proved to be right. 
- It also built confidence in the ability to use the embedded space created by such an AE model meaningfully to create characters along the lines of what we desire and have the model fill out the additional details required to create a game-ready NPC meaning human effort can be spent more meaningfully in creating further developing these characters and bringing them to life in a D&D campaign than on mundane data entry. 
- I believe a transformer could also create great D&D NPCs given the right training set although this would potentially be at the cost of being able to customise them as well as one can with the latent space. Of course, it is possible to create an OpenAI like transformer that makes up for this deficiency too, but I don't believe D&D Character Creation to warrant the spending of energy, resources, and wealth at that scale xD







