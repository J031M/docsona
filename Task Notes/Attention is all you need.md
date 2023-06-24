Let’s start with talking about why we need transformers

It starts with Recurrent Neural Networks. These were sequence to sequence networks. Over here it learnt to map from x to y and after each time step it takes this knowledge for the next time step task. This way previous knowledge of the model is maintained. The image is self explanatory.

![[img1.png]]

problems with RNN :

1. slow computation for long sequences : if n input tokens then n time steps
2. cannot parallelize the process : since previous time step knowledge is needed.
3. vanishing or exploding gradients : during back propagation the gradient might become very small or very large
4. Difficulty in accessing information from long time ago : the longer the network the information from before time step might vanish.

The transformer model solves most of this problems.


![[img2.png]]
the left part is the encoder of the model and the right part is the decoder part of the model. Here data is not split anytime rather every time the arrow splits into multiple heads it means the input data is getting replicated that many number of times. the encoder is replicated Nx times and the output of the last encoder is input to the decoder which might also be replicated many times and the output of the last decoder is the final output.

### encoder
![[img3.png]]

the first part of a transformer is the encoder.

Its first part is the input embedding. Let’s assume the input to the encoder is a set of tokens. each token is given an input ID. the input ID is its position in the vocabulary which is then mapped to a vector of size 512 dimensions. this vector is called embedding. Here for same words the embedding remains the same.

for e.g. : the input tokens are {”your”,”cat”,”is”,”a”,”lovely”,”cat”}. here the token cat will have the same input embedding at both times that it appears.

The next part is the positional encoding. Previously we prepared an input embedding. this however has no knowledge about the position of the token in the sentence. For this we use positional encoding. This provides the model to treat words that appear close to each other as “close” and distant words as “distant”. So we take a new vector that depends only on the position of the token. this will be called position embedding. _[remember this vector depends only on position of token ie if another token was present in same location the other token would have this same position embedding while the original token will have a different embedding that depends on its new position.]_ So we can precompute a set of position embedding vector and using it repeatedly in future. This position embedding will be added to our previous process embedding to prepare our vector that will be the encoder input. the size of this final will remain 512 dimensions.
![[img4.png]]

the functions to calculate the positional encoding is (i is dimension on the vector):
![[img5.png]]

Now the next part of the encoder ie the multi head attention. Let’s first see what’s self attention.
![[img6.png]]

Self-Attention allows the model to relate words to each other ie to find some relation b/w words. The Attention value is given by :

Attention(Q,K,V)=(SoftMax(Q*transpose(K)/sqrt(dk)))*V

here Q stands for Query, K is keys and v is values which just stand for input sentence, dk is dimension of embedding ie 512. After applying SoftMax we get the relationship values b/w each word with another word. Self attention will try to learn the interactions of the word with each other. We then multiply with V(words, dim512) to get Attention matrix which has the same size as the input sequence. so this means we have gotten a matrix whose each row is an embedding of that row which captures the meaning, the position of word and also the interaction with other words.

Self Attention is permutation invariant. It requires no parameters. multi head attention does require some parameters. One nice feature of this is that if we do no want some positions to inter-relate, we can set the value of matrix to -infinity before applying the SoftMax to it. The SoftMax will output 0 then. the model will no learn those interactions then. this will be used in decoder.

The internet says the terms Queries, keys and values come from python dictionaries. Let’s say a dictionary of keys and values is maintained. the query vector is multiplied with keys vector values and then upon applying SoftMax we get the relation b/w the query and the key. this way we can get the most related value to it

![[img7.png]]
Now going to multi-head attention. the input to encoder embedding is copied to 4 parts. 3 of these go to multi head attention. the 3 parts become Q, K and V and hence have same size ie seq_dmodel _[seq→ no. of words , dmodel→ model dimension ie 512]_. Now there will be three new matrices namely Wq, Wk and Wv . these are the weight parameters that the model will try to learn. Their dimension is dmodel_dmodel ie 512_512. The weight parameters are multiplied with corresponding input vectors to get Q`, K` and V` whose dimension is seq_dmodel. In multi head attention, we split these three matrices into number of heads along the embedding axis _[ ie split into 4 parts here]._ So each head will see all the words but each word will have only a part of its embedding. so each head will see the full sentence but each head will have a different aspect of the word. The output of each head is calculated by using the attention technique on each ie :

head=Attention(QWq,KWk,VWv)

the multi-head is calculated as :

MultiHead(Q,K,V)=Concat(head1,head2,…headh)W0

The result of this is the output of multi-head attention ie MH-A whose dimensions are seq*dmodel. So this output would have gathered information about vocabulary, position of word and different aspect on interaction b/w words.

This attention can be visualized. This gives us a knowledge of how each head relate words with each other. The following image should show everything done in multi-head attention.
![[img8 1.png]]

The next step is layer normalization ie labelled as add and norm. In layer Normalization, we calculate the mean and the variance for each vector in a batch of data and then normalize each value. Normalizing is along the feature dimension. two parameters gamma(multiplicative) and beta(additive) are introduced to bring in some fluctuation in the data as normalization can make the data restrictive by squishing them b/w 0 and 1. so these parameters that are learnable by the model helps in amplifying values if the network wants to do so.

Now let’s see the decoder.
![[img.png]]

the output embedding and positional encoding works exactly same as that of the encoder. The difference here is that we will be having two vocabularies(from one lang to another) and this takes care of the embedding needed for translation.

The next layer is the Masked Multi head attention. **Our goal is to make the model causal ie the output at a certain position can only depend on the words on previous positions. The model MUST NOT be able to see future words. **_there some simple manipulation of attention vector of encoder done here after SoftMax._ this step copies the output embedding 3 times ie Q,K and V and does the attention process and normalizes these values. Two copies of the output from this step move on to next step. one copy becomes the Value ie V vector for next attention step.

The rest of the steps in Decoder is exactly same as that of the encoder. The output from the Nth encoder after going through the encoding process is sent as input to the next multi head attention layer in decoder. this input is copied 2 times to prepare the Query ie Q and key ie K vectors. the 3rd input vector ie the Value ie V vector is the output from previous masked multi head attention layer. the rest of the steps within the decoder is exactly same as in encoder. the decoder maybe copied ( or executed)) Nx number of times.

To start the decoder after encoder does its job we send it a token “<SOS>” that stands for start of sentence. **this is at TIME STEP 1.** the output of the decoder is fed to the Linear layer. so our output of decoder will have dimensions as (seq, dmodel). we need to map this to a matrix of dimensions as (seq, vocab_size). this is the job of linear layer ie to map each embedding back to the vocabulary that we have. this output is also commonly called Sofits. SoftMax is applied on this output to then choose one word out of the dictionary. We need N inference time steps for training. **the output is appended to previous sequence and then sent to decoder as input for next time step. ie for time step 2 the input will be “<SOS>prev_output”. That’s why the step to decoder input is labelled as “Outputs(Shifted right)” ie the latest output is appended to previous input.** this way we do the training. The choosing of output word is done using the **BEAM SEARCH** technique. The training is more faster and efficient compared to that of RNNs.

At the end of target sentence we have a label “<EOS>” ie end of sentence. Then we calculate the loss function ie Cross Entropy Loss. the whole task is done in one pass unlike that of RNNs.