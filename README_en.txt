# 16:52 04/18/2024 
# INSA-Toulouse, France
# by Nick Tokmantsev

==============================================================================================
==                                                                                          ==
==                                    Matrix Descriptions                                   ==
==                                                                                          ==
==============================================================================================

Abaqus and Pyxel-DIC use different matrix numbering methods.

			                    F = KU

=====    Abaqus (x1,y1,x2,y2...)

|F_u1|         |K_x1_x1    K_y1_x1    K_x2_x1    K_y2_x1    ...    K_xN_x1    K_yN_x1|    |U_1|
|F_v1|         |           K_y1_y1    K_x2_y1    K_y2_y1    ...    K_xN_y1    K_yN_y1|    |V_1|
|F_u2|         |		      K_x2_x2    K_y2_x2    ...    K_xN_x2    K_yN_x2|    |U_2|
|F_v2|    =    |				 K_y2_y2    ...    K_xN_y2    K_yN_y2| @  |V_2|
|... |         |  ...        ...        ...        ...      ...      ...        ...  |    |...|
|F_uN|         |					    ...    K_xN_xN    K_yN_xN|    |U_N|
|F_vN|         |					    ...    K_xN_yN    K_yN_yN|    |V_N|


=====     Python (x1,x2,...,xN,y1,y2,...,yN)

|F_u1|    |K_x1_x1 K_x2_x1 ...  K_xN_x1	  K_y1_x1  K_y2_x1   ...    K_yN_x1|     |U_1|
|F_u2|    | 	   K_x2_x2 ...  K_xN_x2   K_y1_x2  K_y2_x2   ...    K_yN_x2|     |U_2|
|... |    |  ...     ...   ...   ...       ...      ...      ...      ...  |     |...|
|F_uN| =  |		   ...  K_xN_xN   K_y1_xN  K_y2_xN   ...    K_yN_xN|  @  |U_N|
|F_v1|    |		   ...            K_y1_y1  K_y2_y1   ...    K_yN_y1|     |V_1|
|F_v2|    |                ...                     K_y2_y2   ...    K_yN_y2|     |V_2|
|... |    |  ...    ...    ...     ...      ...      ...     ...      ...  |     |...|
|F_vN|    |        		          K_y1_yN  K_y2_yN   ...    K_yN_yN|     |V_N|



==================================    Explanation    =========================================

U Vector:
    This is a displacement vector. Each element of the vector, i.e., each displacement,
    is written as A_i, where "A" denotes the direction of the Degree of Freedom (DOF) (U for 
    the x-axis and V for the y-axis), and "i" is the node where the displacement occurs.
    
    Example: V_420 means that it is the displacement of node 420 in the Y direction.



F Vector:
    This is a reaction force vector. Each element of the vector, i.e., resultant force, is 
    written as F_ai, where "a" represents the direction of DOF (u for the x-axis and v for the
    y-axis), and "i" is the node where this DOF is applied.
    
    Example: F_x35 means that it is the resultant force at node 35 in the X direction.



K Matrix:
    This is a stiffness matrix. Each element of the matrix is written as K_ai_bj, where a and 
    b are the stiffness directions ("x" for X and "y" for Y) and "i"/"j" are the node numbers.
    
    Example: K_x420_y69 means the stiffness between node 420 along the X-axis and the
             node 69 along the Y-axis (with respective displacements).




The numerical values of elements with the same index are identical. The only thing that changes
is the position.



    Example: K_python and K_abaqus are of dimensions mxm, where m > 2*15. 
	     
	     K_x2y15_python == K_x2y15_abaqus, even though 
             
             K_x2y15 in Python is located at the position [2, m/2 + 15] and 
	     K_x2y15 in Abaqus is located at the position [4, 30].







==============================================================================================
==                                                                                          ==
==                               Function Descriptions                                      ==
==                                                                                          ==
==============================================================================================

===================    1/ read_INP(filename)

Input data:
    filename - the name of the .INP file

Output data: a dictionary containing the numbering of elements, noted in the 
    following form (mapping):

                Key Type Size    Value
                 0  list  4   [1, 2, 5, 4]
                 1  list  4   [2, 3, 6, 5]
                 2  list  4   [4, 5, 8, 7]
                 3  list  4   [5, 6, 9, 8]

     This dictionary contains the index (key) of each element and the connectivity order of 
     local nodes in the global system. For example,


                Key Type Size    Value
                 0  list  4   [1, 2, 5, 4]

      Can be described as follows:

            Element number - 0
            The first  local node is at the 1-st global node
            The second local node is at the 2-nd global node
            The third  local node is at the 5-th global node
            The fourth local node is at the 4-th global node
    
      The Type and Size are not important for the rest of the code. They are always 
      "list" and 4
    

Operation:
    The data containing the numbering is found between the lines "*Element, type=CAX4" and 
    "*Nset, nset=Set-1, generate". The function reads the INP file line by line, takes all the
    values found between the lines and transforms them into a dictionary.







===================    2/ read_MTX(filename, t)

Input data:
    filename - the name of the .MTX file
    t        - the number of DOF for each element

Output data: a dictionary in the following form:

                Key      Type         Size     Value
                 0  Array of float64 (8, 8)   [....]
                 1  Array of float64 (8, 8)   [....]
                 2  Array of float64 (8, 8)   [....]
                 3  Array of float64 (8, 8)   [....]

   This dictionary contains the index (key) of each element and a local stiffness matrix. The 
   Type and Size are not important for the rest of the code. They are always "Array of float64"
   and (8,8).
    

Operation:
    The function reads the MTX file line by line, assigning and affecting to each of
    the 4 elements the following variables (in order):
    
                element, row, col, value
    
    With these values, the function creates the local matrix for each element.







===================    3/ assemble_matrice_globale(element_matrices, node_mapping, dof)

Input data:
    element_matrices - the dictionary of local matrices 
    node_mapping     - the dictionary containing the numbering of elements
    dof              - the number of DOF of the whole system 

Output data: the global matrix in the Abaqus notation (x1, y1, x2, y2...)


Operation:
    An empty stiffness matrix of dimensions mxm is created,  m = dof.

    This matrix is filled by calling on the local matrices and the element mapping.
    







===================    4/ rearrange_matrix_abacus_to_python(matrix)

Input data:
    matrix - the global matrix K in Abaqus notation (x1, y1, x2, y2...) 
    
Output data: 
    rearranged_matrix - the global matrix in Python notation (x1, x2... y1, y2...)


Operation:
    The function moves the elements of the matrix according to the Python notation.
    






===================    5/ Others functions
The following functions are used to create Excel files and format them.

    def generate_labels_notation_abaqus(n)
    def save_matrix_to_excel_notation_abaqus(matrix, filename='output.xlsx'):

    def generate_labels_notation_python(n):
    def save_matrix_to_excel_notation_python(matrix, filename='output.xlsx'):
