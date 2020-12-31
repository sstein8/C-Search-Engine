# Project Report

## Implementation
I chose to do option 1 where there are only three documents inside of the p5docs directory.  To begin, in the main method of test.c, the training function is executed.  In the training function, the user is asked how many buckets they would like and then the function creates a hashtable with that many buckets.  It then calls the function called inputFileIntoHashTable on each of the documents in the directory(D1.txt - D3.txt).  This function takes in a pointer to a file and the hashtable as parameters.  It uses the fopen function to open up each document for reading and then uses fscanf to read each word of the file.  For each word that it reads, it inserts it into the hashtable with the function called hash_table_insert.  This is like a normal insertion function for any hashtable.  It uses the function hash_code for each word to determine which bucket to insert the word into.  Hash code takes each word, sums up the ascii values of each letter in the word, and performs a modulo operation with the sum and the number of buckets.

After the hashtable is created, the function stop_word is called.  This function takes the hashtable in as a parameter.  This function needs to remove any words with an idf score of 0.  I used the base version of stop_words, so it does not take into account other types of potential stop words.  I created a for loop that goes to each bucket in the table.  For each bucket, it traverses the list until it reaches the end.  For each node in the list, it performs the function ht_get, which returns the number of occurrences of the word in a specific docuemnt.  The ht_get function is called 3 times per word for each of the 3 documents.  If the ht_get function returns 0 for each of the 3 documents, that means the df score is 0.  If it returns a positive number for only one of the documents and 0 for the other 2, the df score is 1.  If it returns a positive number for 2 of the documents and 0 for the other, then the df score is 2.  If it returns a positive number for all three documents, the df score is 3.  This logic is to determine how many documents a word appears in, so it doesn't really matter what the number of occurrences is, as long as it is a positive number above 0, it appears in the document.  It then calculates the idf score for each word, and it the idf score is 0, it calls the function ht_remove to remove the word.

ht_remove takes in the hashtable and the word as a parameter. If the hash table is not null, it will calculate the bucket that that specific word is located in and move through the bucket until it finds the word.  If it finds the exact match for the word, it removes it.  If the word is the only element in the bucket, it gets rid of it, and if it is not the only element in the bucket, it moves around the next pointer to set it as the pervious element's next elememt, breakeing it out of the chain.  

Calling the stop_word function marks the end of the training phase. 

The next phase requires the user to enter a search query and the program will performs the necessary calculations to see which documents are most relevant to the words entered.  Therefore, the main method will prompt the user for their search string or X.  The program scans in the user's response and separates it by spaces to extract individual words.  If the user enters x, the program exit using the exit() function.  If the user enters anything else, the program calls the read_query function.

The read_query function takes in the hashtable and a pointer to the query.  I use strtok to tokenize the query and separate each word from the spaces between them to extract each word.  A character pointer called pointer points to the tokenized query.  The user can enter up to 100 words in their query, since the array I store the words in has spots for 100 words.  While the pointer is not null, it performs the rank function on each word.  The rank function computes the bucket that each word belongs in using the has_code function, and then searches through each word in the bucket to see if the word from the search query matches any of the words in the given documents.  If the word from the query is the same as the word in the hashtable from D1, the term frequency of the word in document 1 (tf1) is equal to the number of occurrences of the word in document 1.  This is also checked for the other remaining documents.  If the word in the search query does not occur in the any of the documents, the term frequency of that word for each document is 0.  Once we have the term frequencies calculated for each word in the search query, we have to calculate the idf score, so we call the function calculateIDF, which performs the same calculations as in the stop_word function.  Now we have to calculate the tf-idf score of the words for each document.  I just multiply the tf by the idf, as shown in the project description.  Now, for every word in the search query, there is a calculated tf-idf score correspinding to each document in the directory.  We much add each of these scores to an array so we can access them later to rank them. I have a function called addNode which adds each of the 3 tf-idf scores per word to an array.  I have created a struct called tfIDFnode which holds the tf-idf scores of each word.  The addNode function takes the values passed in, assignes them to the node, and inputs the node into an array.  

After the rank function is done being called, still being inside of read_query, another array is created called rank_docs that holds 3 elements.  Next, the function get_ranks is called.  This function takes in the the values at rank_docs[0], rank_docs[1], and rank_docs[2].  At the moment, these are empty.  The, it goes through out other array, word_array and extracts the tf-idf scores for each document for each word and adds all of the tf-idf(1)s (first column) and puts them into rank_docs[0], adds all of the tf-idf(2)s (second column) and puts them into rank_docs[1]...and so on.  This array holds the rank scores.  

Since the ranked documents need to be printed to a text tile called search_scores.txt, the read_query function creates a file pointer called outptr, uses fopen to open up the search_scores file for writing, and uses fprintf to print this information to the file.  These documents need to be printed in order of relevance, and if a document has a score of 0, it shouldn't be printed.  Therefore, I had to use a lot of conditional statemente to figure out how to print the correct order of the documents and their corresponding scores.  My algorithm for this has 2 main if-statements: if the rank value of the firt document is greater than or equal to the rank value of the second document, and if it is not greater than or equal to the rank value of the second.  Within that first if-statement, I check for 3 more conditions: if the rank value of the second document is greater than or equal to the rank value of the second document, if the rank value of the first document is greater than or equal to the rank value of the third document, and it the rank value of the first document is not greater than or equal to the either.  Within the condition if(rank_docs[1] >= rank_docs[2]), I check if the rank values of document 2 and 3 are zero.  In this case, only print the value of the 1st document.  If all 3 are 0, it prints out "No Matches".  Then I check if the value of the 3rd rank score is 0.  In this case, print the values of the first and second documents.  Lastly, if none are zero, print the the values of document 1, docuemnt 2, and document 3 because they are in order from greatest to least value.  In my second conditional statement inside the first main one, where I check else if(rank_docs[0] >= rank_docs[2]), I check if the rank value of document 2 is 0.  If it is, then only print document 1, then document 3.  If none are equal to 0, then print documents 1, 3, 2, since they are in descending order of rank values.  In my last conditional check inside the first main one, it is an else statement, meaning the first rank value is greater than or equal to the second, but not the third, and the second is greater than the third.  If the rank values of document 2 is equal to 0, it prints the value of documents 1 and 3.  If none are equal to 0, it prints the value of document 1, 3, and 2 in that order.

The second main conditional of this algorithm is for when the rank value of the first document is not greater than or equal to the rank value of the second one.  It first checks if the rank score of the first is >= the rank score of the 3rd.  If it is, it checks if the rank scores of document 1 and 3 are 0.  If so, it prints out the score of document 2.  If the rank score of only document 3 is 0, it prints out documents 2 and 1 and their scores.  If none of the rank scores are 0, it prints out documents 2,1, and 3 in that order. The next check is if the rank score document 2 is greater than or equal to that of document 3.  If it is, it checks if the rank score of document 1 is 0.  If it is, it prints out documents 2 and 3 and their scores.  If none are 0, it prints out documents 2, 3, 1 in that order.  Lastly, the third check is an else statement, so if it didn't pass any of the other checks, it checks it the first rank score is 0.  If it is, it prints out documents 3 and 2 and their scores.  If there are no ranks of 0, it prints out documents 3,2,1 and their scores.



### Question (a): If we have N documents, and document Di consists of mi words, then how long does this simplesolution take to search for a query consisting of K words. Give your answer in big O notation. A more efficient solution can be constructed using other data structures. 

The speed of linked list traversal = O(n) for n items in a list.
The linked list must search from document 1 through document N,and if each document has mi words, there are m(1) + m(2)... + m(N) words in total.
This is the summation from 1 to N of (mi).
Since the speed of traversing the list for n items is O(n), this list has (summation from 1 to N of (mi)) items.  Therefore, the time to search for one word in the list is O(summation from 1 to N of (mi)).
In this question, there is a query of K words, so this operation would repeat K times, to give a final time of K(O(summation from 1 to N of (mi)).

### Question (b): If we assume that the hash function maps words evenly across the buckets (i.e., each bucket gets the samenumber of words), then what is the time complexity (big O notation) of this improved solution? Use the same parameters as Question (a) for the size of each document and size of the query set.
Since the hashtable has different buckets rather than one single list, it only needs to search through one of the lists.  Since the total number of items in the whole hashtable is the summation from 1 to N of mi, it there are b buckets, the total amount of items in one bucket is (summation from 1 to N of mi)/b.  Therefore, the time to search for the query in the hashtable would be K(O(summation from 1 to N of mi)/b)).

### Question (c) why does this lead to a more efficient search process ? If you want to build a more realistic system, you can combine this algorithm along with a statically provided stop word list (i.e., common prepositions etc.). You should specify in your project report which option you chose (i.e., base option or the base plus stop word list).
Removing words with an idf score of 0 makes searching through the list more efficient.  As stated in the project description, the stop words in this context are the words that appear in each of the documents, so it would be unnecessary to include them in the hashtable.  There would just be extra words for the program to traverse through, increasing the total time to retreive the other words in each of the functions that require traversal.  If a word appears in each document, then it doesn't have any bearing on deciding which document is more relevant to the user's query, so it is best just to remove them.

### Question 1(d): Which of the two approaches is more efficient in terms of removing stop words and why. Remember, your final project score will depend on how efficient your solution is. Note: You could still have a working project with less than ideal efficiency and will receive some credit (but not 100%) for a working solution. 
The second approach to removing stop words is likely more efficient since an entire list can be removed at once rather than having to search through a list with many different words and individually calculating the idf scores of each word and then having to call a remove function on each word that has to be deleted.  In the second approach, only one calculation needs to be done since all of the words in the list are the same.



## Flow Chart
![IMG_0261](https://user-images.githubusercontent.com/59673538/102155330-5c7a8480-3e49-11eb-8935-669a46f4cff5.PNG)


```
## Makefile
.PHONY: clean build all

# compilation settings
CC = gcc
CFLAGS = -I$(INCLUDE_DIR) -Wall -Wextra -Werror -pedantic -std=gnu99 -pedantic -Wmissing-prototypes -Wstrict-prototypes -Wold-style-definition -g

# directory paths
INCLUDE_DIR = ./
SRC_DIR = ./
OBJ_DIR = ./

# file lists
CFILES = hashtable.c test.c
OBJS = hashtable.o test.o

# binary
BIN = search

$(OBJ_DIR)/%.o: $(SRC_DIR)/%.c
	$(CC) $(CFLAGS) -o $@ -c $<

$(BIN): $(OBJS)
	$(CC) -o $@ $(OBJS)

clean:
	rm -f $(OBJS) $(BIN) *~
```

