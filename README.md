# Welcome to LDA_Gibbs! 
This is my statistics research project about natural language processing with Dr. Nicole Dalzell at Wake Forest University. One of the main concentrations of this project is topic modeling and I focus on latent Dirichlet allocation (LDA) with Gibbs sampling process. LDA is effective at classifying documents into latent topics. After digging into this algorithm, I have finished my own version of [*LDA_Gibbs.Rmd*](https://github.com/tttyuntian/Statistics-Thesis/blob/master/LDA_Gibbs.Rmd) in R. More information of this script will be shown below.

To learn about the theoretical part of LDA, you can read through Section 3 of my thesis paper [*A Brief Introduction to Natural Language Processing.pdf*](https://github.com/tttyuntian/Statistics-Thesis/blob/master/A%20Brief%20Introduction%20to%20Natural%20Language%20Processing.pdf). 

# 1. LDA_Gibbs Introduction
LDA_Gibbs() is an easy-to-use function that can classify documents into latent topics. In the following sections, I use the [Neural Information Processing Systems (NIPS) dataset](https://www.kaggle.com/benhamner/nips-papers) from Kaggle. This dataset contains information of the title, authors, abstracts, and paper contents for all NIPS papers from 1987 conference to 2016 conference.

## 1.1 LDA_Gibbs(...) Function
**LDA_Gibbs(** *corpus, K = 5, control = 1123, burn_in = 200, converge_iteration = 100, threshold = 5e-3, max_iteration = 1000, stop.words = NULL* **)**

Implement LDA with Gibbs sampling process.

Return a list containing beta data.frame, theta data.frame, and theta_param list.

* **Parameters**
	* **corpus: *corpus data.frame***<br>
		A data frame with multiple rows and one column. Each row represents one document.
	* **K: *None or int, optional***<br>
		The number of latent topics. If there is no input, then *K = 5* by default.
	* **control: *None or int, optional***<br>
		The seed for pseudo-randomness. If there is no input, then *control = 1123* by default.
	* **burn_in: *None or int, optional***<br>
		The number of burn-in iterations before the function starts tracking the values of theta parameters.<br>
		If there is no input, then *burn_in = 200* by default.
	* **converge_iteration: *None or int, optional***<br>
		The number of iterations needed to ensure that the algorithm has converged.<br>
		If there is no input, then *converge_iteration = 100*.
	* **threshold: *None or float, optional***<br>
		The converge threshold for increase/decrease rate of a theta parameter. <br>
		If increase/decrease rate is less than the threshold for *converge_iteration* times continuously, then the algorithm has converged.<br>
		If there is no input, then *threshold = 5e-3*.
	* **max_iteration: *None or int, optional***<br>
		The maximum number of iterations of Gibbs sampling process.
	* **stop.words: *None or list, optional***<br>
		The list of stop words that should be ignored in the corpus.<br>
		If there is no input, then *stop.words = NULL*.
* **Returns**
	* **res: *A list containing three results***
		* **beta: *Tidy table***<br>
			A tidy table with  three columns, including *topic*, *word*, and *beta*.<br>
			*topic* indicates the identification of the latent topics. <br>
			*word* indicates the words. <br>
			*beta* indicates the probability of a specific *word* falling into a specific *topic*.
		* **theta: *theta matrix in tidy format***<br>
			A tidy table with  three columns, including *document*, *topic*, and *theta*.<br>
			*document* indicates the identification of the documents.<br>
			*topic* indicates the identification of the latent topics.<br>
			*theta* indicates the probability of a specific *document* falling into a specific *topic*.
		* **theta_param: *List***<br>
			A list of theta parameters of a chosen document over iterations. You may use this to check whether the algorithm has converged.

## 1.2 Example with NIPS Dataset
### 1.2.1 Connect and Sample from Dataset
After downloading [NIPS dataset](https://www.kaggle.com/benhamner/nips-papers), move `database.sqlite` and `LDA_Gibbs.Rmd` under the same folder. Then, in R, edit `LDA_Gibbs.Rmd` and append the following code.

    library(RSQLite)
    # Connect to database
    db <- dbConnect(dbDriver("SQLite"), "database.sqlite")
    # Sampling from the database
    papers_sample <- dbGetQuery(db, "
	SELECT paper_text
	FROM papers
	WHERE year > 2015
	ORDER BY id DESC
	LIMIT 10")

`papers_sample` is a *data.frame* with 1 column and 10 rows, where each row represents a paper.

### 1.2.2 Implement LDA_Gibbs(...) with `papers_sample`
Now, we have `papers_sample` as our corpus. Say we have the following list of stop words and call LDA_Gibbs() with these parameters. This might take a while.

    # List of stop words
    stop.words = c("0", "1", "2", "3", "4", "5", "6", "7", "8", "9", "model", "algorithm", "models", "learning")
    # Call LDA_Gibbs()
    res = LDA_Gibbs(papers_sample, K = 3, control = 1123, burn_in = 200, converge_iteration = 100, threshold = 5e-3, 
    		max_iteration = 1000,  stop.words = stop.words)

LDA_Gibbs() will output its progress and whether it has found convergence as shown below.

	[1] "Start initializing..."
	Joining, by = "word"
	[1] "Done Initializing."
	[1] "Start burn in process..."
	[1] "Done burn in process."
	[1] "Tracing Document #9"
	[1] "Start looking for convergence..."
	[1] "Convergence Found at Iteration 356."
	[1] "Done looking for convergence."

`"Convergence Found at Iteration 356."` means that the increase/decrease rate of theta values is less than threshold (i.e. 5e-3) for 100 iterations continuously, so LDA has converged.

Now, `res` stores the LDA results. We can extract each result by using following code.

    # Extract result from res
    beta = res$beta
	theta = res$theta
	theta_param = res$theta_param

### 1.2.3 Check Convergence
Before we start interpreting the results, we need to make sure that LDA has converged using `theta_param`. LDA_Gibbs() function will randomly pick one of the documents from the corpus and keep track of its theta values in `theta_param`. Therefore, to check convergence, we can use the following code.

	# Check for convergence
	plot(theta_param$topic1, type = "l", xlab = "Iteration", ylab = "theta")
	
![](https://github.com/tttyuntian/Statistics-Thesis/raw/master/art/convergence.png)
	
Since the convergence is found at Iteration 356, we can see that theta values do not fluctuate too much after 250th iterations. Therefore, we conclude that LDA has converged.

### 1.2.4 Interpret LDA Results
To understand the results, we can look into one document and see which topic it is classified as.

    library(tidyr)
    # Show the probability for a document falling into the latent topics
	document_index = 1
	theta %>%
	  filter(document == document_index)
	 
The return will be:

	document	topic		theta
	<int>		<chr>		<dbl>
	1		1		0.000545561		
	1		2		0.001617088		
	1		3		0.997837351	

This result means that *Document 1* is most likely to be classified as *Topic 3*. To understand what *Topic 3* is, we need to look into it by using following code.

	# Select the top 20 words with highest beta values in this topic
	topic_index = 3
	beta %>%
	  filter(topic == topic_index) %>%
	  arrange(desc(beta)) %>%
	  top_n(20)

The return will be:

	topic		word		beta
	<int>		<chr>		<dbl>
	3		networks	0.006004027		
	3		log		0.005935525		
	3		clustering	0.004883109		
	3		lds		0.004640842		
	3		feedback	0.004304822		
	3		matrix		0.004199121		
	3		user		0.003755266		
	3		node		0.003704164		
	3		time		0.003648625		
	3		data		0.003581415	
	3		queries		0.003048854		
	3		algorithms	0.003017439		
	3		s0		0.002862436		
	3		estimation	0.002773142		
	3		al		0.002759629		
	3		probability	0.002669478		
	3		xt		0.002650911		
	3		network		0.002613383		
	3		graphon		0.002580034		
	3		likelihood	0.002561868	

From these words, we can relate *Topic 3* to clustering, lds (i.e. Linear Dynamical System), and matrix. Let's peek at the first few lines of the abstract of *Document 1*, "Multi-view Matrix Factorization for Linear Dynamical System Estimation". 

	papers_sample[1,]

> We consider maximum **likelihood** **estimation** of **linear dynamical systems** with generalized-linear observation models. Maximum **likelihood** is typically considered to be hard in this setting since latent states and transition parameters must be inferred jointly. Given that expectation-maximization does not scale and is prone to local minima, moment-matching approaches from the subspace identification literature have become standard, despite known statistical efficiency issues. In this paper, we instead reconsider **likelihood** maximization and develop an optimization based strategy for recovering the latent states and transition parameters. Key to the approach is a two-view reformulation of maximum **likelihood** **estimation** for **linear dynamical systems** that enables the use of global optimization **algorithms** for **matrix** factorization. We show that the proposed **estimation** strategy outperforms widely-used identification **algorithms** such as subspace identification methods, both in terms of accuracy and runtime.

I highlight the words that also show up in top 20 words with largest beta values. This shows LDA_Gibbs() works. You can now play with NIPS dataset. **One friendly reminder is that LDA generally needs a larger corpus, even though we only use 10 papers and it works in this example.**

# 2. Upcoming Updates
[03/17/2020] I will keep polishing and updating LDA_Gibbs(). Currently, I am digging into infinite latent variable model and try to implement it into LDA_Gibbs() so that LDA_Gibbs() can help us find an optimal number of latent topics.