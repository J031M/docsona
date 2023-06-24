- This presents method LLM.in8() for 8-bit matrix multiplication in transformers which maintains performance of 16-bit precision models.
- it uses following two approaches:
	- vector-wise quantizatioin to quantize most features
	- mixed-precision decomposition for outlier(16 bit multiplication), for the rest(8 bit multiplication)
- it shows that LLM.int8() method maintains the performance of 16-bit models for transformers up to 175B parameters, while almost using half of its memory(50%).
- Outlier features used not so frequently even so in a consistence manner. Its occurring small number of dimensions. But, removing the outlier features significantly degrades models' performance.
- process: 
	1. The method uses vector-wise quantization, where a different scaling constant is assigned to each row and column for the matrix multiplication. This helps increase precision compared to using a single scaling constant.
    
	2. In addition to vector-wise quantization, the method uses mixed-precision decomposition. It identifies outlier features with large magnitudes in the model and multiplies those in 16-bit floating point. All other values are multiplied in 8-bit.
    
	3. The 8-bit vector-wise multiplication is done by scaling the inputs by the row and column-wise absolute maximum values. The outputs are then quantized to 8-bit integers.
    
	4. The 8-bit integer matrix multiplication outputs are dequantized by the outer product of the row and column scaling constants.
    
	5. The 16-bit and 8-bit multiplication outputs are then accumulated in 16-bit floating point to get the final result.