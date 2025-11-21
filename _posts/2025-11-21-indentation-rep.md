---
title: 'Simple Perception: Indentation Representations '
date: 2025-11-21
permalink: /posts/2025/11/indentation-rep/
tags:
  - mech interp
  - representation
  - perception

---

Revealing how a model represents a "visual" feature: indentations. 

Simple Perception: Indentation Representations
======

Background
------
This work was inspired by the Gurnee, et al. paper, "When Models Manipulate Manifolds: The Geometry of a Counting Task" [1] where they highlight an interesting and important perspective for interpretability. Unlike the studies looking into the mechanisms underlying higher level cognitive processes, they instead consider the less semantic and more perceptual features that the model might represent. This is easier to understand when dealing with a visual model where the input will have clear and distinct perceptual features, but for a language model that is less obvious. 

What might these perceptual features be in a language model? Here I look into indentations as they can operate as signposts when viewing a block of text. While they could be written off as generic whitespace, they could also carry important information that tells the model where it is in a sequence, particularly for coding. Indentations are key to writing proper functions, therefore understanding where you are in the nested structure would be highly important for producing logical, and functional, code. This work focuses on the representational structure for indentations in sample code using unsupervised methods.

Key Takeaways
------
1. Different levels of indentations are distinctly represented in earlier layers of the model. 
2. Additional context decreases the similarity between the indentation representations. 
3. Indentations are represented ordinally with each representation being most similar to its neighbor.
4. PCA analysis reveals a clean Fourier-like representation of the indentation levels.

Results
------
<p>As this study focuses on the representations of indentations in code, I sought out an open-source model that had coding capabilites and landed on Qwen2.5-7B. A first check revealed that while different levels of indentations were represented by unique tokens, they only had unique representations throughout the model layers when following a newline character (Fig. 1A). Therefore, all of the prompts used for these analyses included a newline character prior to the indentation (Fig. 1B). Additionally, with more context the model further distinguishes the representations of the indentation tokens, though the trend is similar with the strongest distinction in the earlier layers (5-10), even when the indentations are presented within a variety of code contexts (Fig. 1C). This suggests that these layers could be important for encoding the indentation information, though the representations are not completely dissociated from semantics. For code, this makes sense as the specific line of code will be informative as to whether the next line needs to be indented at all. Regardless, I zoomed into layer 7 in particular to investigate this further.   
</p>

<img src="/images/bp2/Fig1.png" alt="Figure 1" width="75%" height="75%">
<p>
<em><b>Figure 1: Indentation representational similarity.</b> (A) Minimum cosine similiarity between indentation representations with and without the newline character. (B) Generated prompts for the isolated indentations, control context, and randomized context. (B) Minimum cosine similarity between indentation representations within different contexts. </em>
</p>
<p>Indentations indicate where in a nested bit of code you are located, following an ordinal path as the indentations become longer. Therefore, indentations that are closer together in distance are typically closer together in the code, making an argument for those representations to potentially be more similar than indentations further apart. This is in fact the case, as the similarity between the representation of each indentation level is highest with its immediate neighbors and falls off with increased distance (Fig. 2). This produces a clear negative correlation between distance and similarity, following a linear trajectory (Fig. 2B). This is solely in layer 7 as that layer showed the lowest minimum cosine similarity, however this trend holds across many layers.  
</p>

<img src="/images/bp2/Fig2.png" alt="Figure 2" width="75%" height="75%">
<p>
<em><b>Figure 2: Similarities between indentation representations are negatively correlated with difference in indentation length. </b> (A) Cosine similarity matrix for indentation levels one to ten in layer 7. Representations are more strongly correlated with their neighbor level and fall off as the distance increases. (B) Difference between each indentation level for a given pair of indentations plotted against their cosine similarity.  
</em>
</p>
<p>Correlations between distance and similarity remain strongly negative across all of the layers in the model, so this ordered structure is maintained even in the later layers. Although the slopes of the linear fits follow along with the trends seen in the minimum cosine similarity plots with the largest slope in layer 7 and then it steadily decreases. While the correlations remain negative the spread of the differences in representations are largest in the earlier layers. 
</p>

<img src="/images/bp2/Fig3.png" alt="Figure 3" width="75%" height="75%">
<p>
<em><b>Figure 3: Distance and similarity correlations across all layers. </b> (A) Measured correlations between each indentation pairs' distance and similarity at each layer. (B) The slope of the linear fit for this relationship across all layers.  
</em>
</p>
<p>I then turned to PCA to try and unpack these representation relationships further. First, zooming in again on layer 7, a large portion of the variance was explained with the first two PCs (66.8%). Plotting those we see that the there is a precise structure to the representation of each of these indentation levels in PC space with the first PC following a linear trajectory and the second becoming parabolic (Fig. 4AB). Given that there are a few layers around layer 7 that have low cosine similarities, I expanded to look at layer 5 through 9, and also included the third PC for a more complete picture. We see that the PC space is broken out into a Fourier-like structure with the third PC following a sinusoidal pattern across the indentation levels (Fig. 4C). These patterns are steady across these 5 layers with small differences in magnitude, but this changes fairly significantly in the layer layers. In the later layers this strucutre begins to break down particularly at high indentation levels. As these later layers are likely incorporating more context in their representations there could be a handful of reasons why this is happening. One proposed idea is that these higher indentation levels are not as common in coding and therefore it is unnatural for these later layers. In contrast, the early layers may not need to have that context to encode these indentations and therefore they keep a periodic representation. 
</p>

<img src="/images/bp2/Fig4.png" alt="Figure 4" width="65%">
<p>
<em><b>Figure 4: Geometric representation of the indentation levels within and across layers.</b> (A) First principal component of the indentation representations in layer 7. (B) First and second principal components in layer 7. (C) Expanded PCA, including PC1, 2 and 3, for layers 5 through 9. Red is associated with the highest layer, indigo with the lowest. (D) Expanded across all layers. Red is the last layer, indigo is the first, following reverse rainbow order for the rest. </em>
</p>
<p>To causally connect this head to the bias circuit a series of ablations were carried out targeting different levels involving this attention head to see how specific the ablation could be and still be effective. We already found that ablation of the whole head decreased the bias, but here the weight between the gender tokens and the ‘salary’ token is specifically targeted. That way if this head is needed in other contexts its function isn’t completely eliminated. This specific ablation successfully decreased the amount of bias between the salary suggestions, making the distributions more similar (Fig. 5A,B).
</p>
  
Conclusions, Limitations, and Future Directions
------
In this work we find that indentations are distinctly represented in the model. While they do rely on the presence of a newline character to define them properly, the levels of the indentations are represented in a clean and structure manner in the earlier layers of this model. The model maps the distance between each indentation length across three dominant PCs, each with their own periodic behavior. This way, in a coding environment, the model would be able to align its location in the nested structure using the indentation levels and their relative similarities. However, while this preliminary work unveiled this encoding structure, ongoing and additional work is needed to see whether the model operates on these representations causally. Not unlike in neuroscience, observational studies and dimensionality reduction techniques can be helpful for us to grasp how a system might make sense of and organize perceptual information, but whether the system is using that structure explicitly is a different question. The goal of the next blog post is to dive into causal analyses, perturbing this structure and these top dimensions to then measure the effect on prediction accuracy. An additional interesting future pursuit is to investigate how the global structure is represented, probing a full code block rather than a few single lines. All together, this work supports the idea that there are natural perceptual inputs that even a pure language model encodes. Therefore, it will be fruitful to investigate these features alongside the higher cognitive processes. 

References
------
[1] Gurnee, et al., "When Models Manipulate Manifolds: The Geometry of a Counting Task", Transformer Circuits, 2025.

