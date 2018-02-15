# some notes for deep learning parameter tuning

* for fully connected models depth usually does not matter. 
* for Convolutional models depth has a significant impact on the model performance as these models learn the different features through a hierarichal structure and thus more the number of layers more the features that can be learned.
* Large Networks may overfit, but that does not mean that we should small networks because small networks usually are difficult to train with methods like gradient descent.
* order in which you should use actications: ReLU, MaxOut, LeakyReLU, tanh, and on your own risk (Sigmoid). (ReLU may die during the gradient descent updates(Be careful!!).)
* To avoid overfitting use the regularization 

# some Data preprocessing Techniques
1. center around mean
	Two ways are usually preferable: 
	1. Subtract a single value from all the data
	2. do it for every channel in case of image recognition
2. Normalization
	This is a technique to scale each value to same scale so that it becomes easier to do a gradient descent.
	1. Dividing by the standard deviation along each dimension.
	2. Divinding so that the min and max are in the range [-1, 1]
	*Note: Usually it is not a problem in case of Image Recognition*
3. PCA and whitening
	can be used to reduce the size of the data or number of features.

# Weight Initializations
1. Zero Initialization:
	Not a good option at all
2. Small numbers
	A Good Idea...
	e.g. `0.01 * np.random.randn(D, H)` use small values centered around zero.
	may not work always because the gradient signal may dimnish during backpropagation and is a concern for deeper networks
	either `gaussian` or `uniform` distribution will work out.
3. Xavier intialization:
	using a weight intialization with 0 mean and var(1/n_in) (n_in = fan in of the layer) 
	references: [Xavier Initialization A good article](https://isaacchanghau.github.io/2017/05/24/Weight-Initialization-in-Artificial-Neural-Networks/#Xavier-Initialization)
	1. To keep up with the backpropagation use variance (2 / (n_in + n_out)) and in code sqrt of the given value 
	__Note__: Don't use the Xavier intialization with ReLU as the gradient won't Explod or shrink with telling...
4. ReLU intialization: Proposed Because the In some deep Neural Networks they have difficulty converging for Gaussian Distribution
	1. Typically used for ReLU activations because they have a non zero mean and thus the traditional methods usually don't work well.
	2. 0 mean and sqrt(2 / n_l(number of neurons in layer l))
5. Sparse Initialization

# bias intialization
1. Usually they are intialized to zero because the asymmetry is provided by the randomly initialized weights
2. For ReLU some people use a small constant value such as 0.01 so that all neurons fire up from the very beginning

# Batch Normalization 
1. Takes care of the ugly intializations
2. In practice it is done by inserting a BatchNorm layer between the Fully connected layer and the activation layer.
3. Batch Normalization on activation output of the previous layer does zero centering by subtracting mean and divinding by the standard deviation of the batch.
4. This is undone by the next layer and so are learnable parameters namely a __standard_deviation__(denoted by *gamma*) factor which is multiplied by the normalized values and __mean__(denoted by the *beta*) value which is added to the the result of previous calculation
5. --(..Yet to be confirmed..)Main thing to know about it is that it is done for every MINI BATCH.
6. Enables higher learning rates

## Why Batch Norm Layer instead of directly evaluating on the activations?

This is because in the latter case the pre-trained weights will be modified.
While as discussed above adding a layer of Batch norm only introduces two new parameters which are learnable params.

## Regularization
Mainly provides a way to reduce overfitting

1. L2 Regularization:
	* Diffused weight vectors instead of highly peaked weight vectors.
2. L1 Regularization:
	* uses the linear weight instead of the squared term
	* Intruging property that it leads the weight vectors to become sparse during optimization and nearly invariant to 'noisy' inputs
__Note__: L2 is better performer over L1 usually.
3. MaxNorm Regularization: 
	* Enforces an upper bound on the values of the weight time the performance is not affected.
4. Inverted Dropout and Dropout
*Interested*: __DropConnect__ is a concept where instead of dropping connections the weights are masked to zero value.

# Bias Regularization
__not required__
	if done may lead to worsening of the results.

__Recommendation__: use L2 regularization combined with Dropout after all convnet layers.


# Loss Functions

__Classification__:

* SVM loss -- max(0, sum(y_j - y_i + 1)) where y_i is the actual answer
* Hinged Loss -- (Sometimes people report it to give better performance)
	* uses squared value(sum((y_j - y_i + 1) &ast;&ast; 2))
* Softmax converts to probabilities in the range of 0 - 1
	* L_i (Loss for ith input) = (-log(exp(f_i)/sum(f_j)))
	* A finer version for large labels is __Hierarichal Softmax__.
	__Attribute Classification__:
	* as in case for tags, wether a label is present for one class or not? L_i = sum(1 - (y_ij * f_j))
	* alternative is to use a regression classifier for each class if it present or not.

__REGRESSION__:
* L2 loss
* L1 loss
* Softmax Loss
*Note*: It is more difficult to optimize the L2 Less as compared to the Softmax loss because in L2 the actual values are used while in softmax the actual values hardly play any important role.
* L2 is prone to outliers

# Some parameters to look for while training

1. if there is a subtle difference between the training/validation accuracy then more regularization or collect more data.
2. If validation accuracy tracks the training accuracy very well then there is a chance that models capacity is low and you must increase the size.
3. Annealing of the learning  rate is also important
	* Step decay
	* Exponential Decay
	* 1/t decay

# Hyper Parameter tuning
1. user learning rate raised to some power e.g 10 ** uniform(-6, 1)
2. Make sure that the values not on the edges and also considering the intermidate values.
3. Perform a coarse to grain search e.g. start from a range a then reduce that range as the you start to obtain results.
4. Bayseian Hyper Parameter Optimization
5. __Model Ensemble__: Train multiple models and at test time use the average of all those
	* Performance usually improves with increasing the number of ensembles
	* __Some strategies are:__
		* __Same Model different random intializations__(usually not a recommended approach)
		* __Top few models determined during cross-validation__(gives variety to ensemble)
		* __Different checkpoints of a single network__(May but usually not a recommended approach)
		* __Averaging over last few weights of the training__(usually always works)


# Some tips and tricks to improve your Hyper Parameters([from the paper by Yoshua Bengio](https://arxiv.org/pdf/1206.5533v2.pdf))
## During training
1. __learning rate__: 
 	* for typical values b/w(1/0) of input learning rate is usually less than 1 and greater than 10^-6
 	* default value(0.01) 
 	* best to be optimized in case of __SGD__
 	* decay rate of (1/(t^alpha))(alpha usually 1)
 		* use alpha < 1 in case of non-convex problem
 2. __Mini Batch Size__:
 	* usually b/w 1 and a few hundreds
 	* a default value of 32 works
 	* tune this after other parameters have been selected
 	* Reoptimization at the end is necessary
 3. Number of iterations
 	* may vary
 	__need to learn more__
 4. momentum
 	* default value of 1
 5. Layer specific hyper parameter optimization can be tried.

## Model Criterion:
1. Hidden Units:
	* Large enough values is required but much larger than enough is not a problem except for computational expenses.
	* For same size per layer usually works better or same as that for a decreasing size
2. Weight Decay:
	* regularization penalty should also mutiply by (1/ number of updates needed to go from once throught the dataset (B / T))
	* for batches win which the last batch has a reduce size term by (B'/B)
	* for pure online learning setting use *B / t* where t is the t-th example
	* L2 and early stopping have the same effect while earlystopping is much more efficient to L2 and thus L2 may be dropped.
	* L1 us different and may sometimes act for feature selection. It makes sure that the parameters that are not important should be driven to zero.
	<!-- *  -->