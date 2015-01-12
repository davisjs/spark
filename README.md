Project Schedule

Our spark code should contain three main parts:

1. construct a rdd-based matrix

i. This part first need to specify a row partitioner and a column partitioner, and then loads data from files or other source to construct a rdd-based matrix. A rdd-based matrix should be a row-block-based matrix or a column-block-based matrix or both.

ii. The next job is to decide how to store a split matrix block in the memory of one computer, which could be a row block or a column block. For example, if it is a row block, which has partial rows and entire columns, it would split the block along columns into many square-like smaller block with partial rows and partial columns.

iii. Then, how should we store a smaller block? There are three ways: (1) we store the block as a small dense matrix, which means we treat the entire matrix as a dense one; (2) If the entire matrix is very sparse and the small block has many all-zeros columns (or rows), we store each row (or column) by column indices (or row indices) and their corresponding values in order. We use a row (or column) array to store all such row (or column). For example:

(4  9  0  0  3  0;
 5  0  0  0  2  0)
 
==> stored as

[0] -> ((0,4) (1,9) (4,3))
[1] -> ((0,5) (4,2))  

(3) If the small block has many all-zeros columns and rows, we store each non-zero entry by its corresponding row index and column index and its value. For example:

(4  9  0  0  3  0;
 5  0  0  0  2  0;
 0  0  0  0  0  0;
 2  0  0  0  0  0)
 
==> stored as

((0,0,4), (0,1,9) (0,4,3) (1,0,5) (1,4,2) (3,0,2))

2. create a small look-up table

A column-to-rowblock look-up table is a small matrix, each entry of which is a bool value meaning whether a column has at least one non-zero entry in a row block. If we want to create a column-to-rowblock look-up table, we should built it from column-block-based matrix; otherwise, to create a row-to-columnblock look-up table, we should build it from row-block-based matrix.

3. do matrix multiplication by efficient join

Because the two rdds have the same partitioner, the join operation is very efficient. The whole process needs two join operations. First join uses (1) the column-to-rowblock look-up table of the left matrix and (2) the row-block-based of the right matrix as two inputs to compute reduce row blocks of the right matrix for sending to the  row-block-based matrix of the left matrix. Second join computes the multiplication and get a new row-block-based matrix.
