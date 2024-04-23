
# 16:52 04/18/2024 
# INSA-Toulouse, France
# by Nick Tokmantsev

==============================================================================================
==                                                                                          ==
==                                  Description des matrices                                ==
==                                                                                          ==
==============================================================================================

Abaqus et Python/Pyxel-DIC utilisent des m�thodes de disposition des matrices diff�rentes.

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

|F_u1|    |K_x1_x1 K_x2_x1 ...  K_xN_x1	  K_y1_x1  K_y2_x1   ...    K_yN_x1|    |U_1|
|F_u2|    | 	   K_x2_x2 ...  K_xN_x2   K_y1_x2  K_y2_x2   ...    K_yN_x2|    |U_2|
|... |    |  ...     ...   ...   ...       ...      ...      ...      ...  |    |...|
|F_uN| =  |		   ...  K_xN_xN   K_y1_xN  K_y2_xN   ...    K_yN_xN| @  |U_N|
|F_v1|    |		   ...            K_y1_y1  K_y2_y1   ...    K_yN_y1|    |V_1|
|F_v2|    |                ...                     K_y2_y2   ...    K_yN_y2|    |V_2|
|... |    |  ...    ...    ...     ...      ...      ...     ...      ...  |    |...|
|F_vN|    |        		          K_y1_yN  K_y2_yN   ...    K_yN_yN|    |V_N|



==================================    Explication    =========================================

Vecteur U:
    C'est un vecteur de d�placement. Chaque �l�ment du vecteur, soit chaque d�placement,
    est �crit A_i, o� "A" repr�sente la direction de DDL (U pour l'axe x et V pour l'axe y) et
    "i" est le n�ud o� le d�placement a lieu.
    
    Exemple: V_420 signifie que c'est le d�placement du n�ud 420 dans la direction Y.



Vecteur F:
    C'est un vecteur de force de r�action. Chaque �l�ment du vecteur, soit force r�sultante, 
    est �crit F_ai, o� "a" repr�sente la direction de DDL (u pour l'axe x et v pour l'axe y) et
    "i" est le n�ud o� ce DDL est appliqu�.
    
    Exemple: F_x35 signifie que c'est la force r�sultante au n�ud 35 dans la direction X.



Matrice K:
    C'est une matrice de rigidit�. Chaque �l�ment de la matrice est �crit K_ai_bj, o� a et b 
    sont les directions de rigidit� ("x" pour X et "y" pour Y) et "i"/"j" sont les num�ros des
    n�uds.
    
    Exemple: K_x420_y69 signifie la rigidit� entre le n�ud 420 suivant l'axe X et le n�ud 69 
	     suivant l'axe Y (avec des d�placements respectifs).



Les valeurs num�riques des �l�ments ayant le m�me indice sont identiques. La seule chose qui 
change est la position. 

    Exemple: K_python et K_abaqus sont de dimensions mxm, m>2*15
	     K_x2y15_python == K_x2y15_abaqus, m�me si 
	     K_x2y15 sur python se trouve dans la position [ 2 ,m/2+15] et
	     K_x2y15 sur abaqus se trouve dans la position [ 4 ,  30  ] 












==============================================================================================
==                                                                                          ==
==                               Description des fonctions                                  ==
==                                                                                          ==
==============================================================================================

===================    1/ read_INP(filename)

Donn�es d'entr�e :
    filename - le nom du fichier .INP

Donn�es de sortie: 
    node_mapping - un dictionnaire contenant la num�rotation des �l�ments
    number_dof   - le nombre des DDL du syst�me (nombre des noeuds multipli� par 2)

    Le contenu du node_mappings est not� sous forme suivante (mapping):

                Key Type Size    Value
                 0  list  4   [1, 2, 5, 4]
                 1  list  4   [2, 3, 6, 5]
                 2  list  4   [4, 5, 8, 7]
                 3  list  4   [5, 6, 9, 8]

    Cette biblioth�que contient l'index (key) de chaque �l�ment et l'ordre de connectivit�
    des n�uds locaux dans le syst�me global. Par exemple,

                Key Type Size    Value
                 0  list  4   [1, 2, 5, 4]

    Peut �tre d�crit de mani�re suivante :

            Num�ro de l'�l�ment - 0
            Premier   noeud local est au 1-er noeud global
            Second    noeud local est au 2-�me noeud global
            Troisi�me noeud local est au 5-�me noeud global
            Quatri�me noeud local est au 4-�me noeud global

    Les Type et Size ne sont pas importants pour la suite du code. Ils sont toujours "list" et
    4.
    

Fonctionnement:
    Les donn�es contenant la num�rotation se trouvent entre les lignes "*Element, type =CAX4" 
    et "*Nset, nset=Set-1, generate". La fonction lit le fichier INP ligne par ligne, prend 
    toutes les valeurs qui se trouvent entre ces lignes et les transforme en une biblioth�que.






===================    2/ read_MTX(filename, t)

Donn�es d'entr�e :
    filename - le nom du fichier .MTX
    t        - le nombre de DDL de chaque �l�ment

Donn�es de sortie: une dictionnaire de forme suivante:

                Key      Type         Size     Value
                 0  Array of float64 (8, 8)   [....]
                 1  Array of float64 (8, 8)   [....]
                 2  Array of float64 (8, 8)   [....]
                 3  Array of float64 (8, 8)   [....]

    Cette biblioth�que contient l'index (key) de chaque �l�ment et une matrice de rigidit� 
    locale. Les Type et Size ne sont pas importants pour la suite du code. Ils sont toujours
    "Array of float64" et (8,8).
    

Fonctionnement:
    La fonction lit le fichier MTX ligne par ligne en assignant et affectant � chacun des 4 
    �l�ments variables suivantes (dans l'ordre) :
    
    		            element, row, col, value
    
    Avec ces valeurs, la fonction cr�e la matrice locale de chaque �l�ment.







===================    3/ assemble_matrice_globale(element_matrices, node_mapping, dof)

Donn�es d'entr�e :
    element_matrices - la biblioth�que des matrices locales
    node_mapping     - la biblioth�que contenant la num�rotation des �l�ments
    dof		     - le nombre des DDL du syst�me 

Donn�es de sortie: la matrice globale dans la notation Abaqus (x1,y1, x2,y2...)
    

Fonctionnement:
    On cr�e une matrice de rigidit� vide de dimensions mxm, m = dof

    On remplit cette matrice en faisant appel aux matrices locales et au mapping des �l�ments.








===================    4/ rearrange_matrix_abacus_to_python(matrix)

Donn�es d'entr�e :
    matrix - la matrice globale K en notation Abaqus (x1,y1, x2,y2...)
    
Donn�es de sortie:
    rearranged_matrix - la matrice globale en notation Python (x1,x2...y1,y2...)
    

Fonctionnement:
    La fonction d�place les �l�ments de la matrice selon la notation Python.





===================    5/ autres fonctions
Les fonctions suivantes sont utilis�es pour cr�er les fichiers excel et une mise en page.

    def generate_labels_notation_abaqus(n)
    def save_matrix_to_excel_notation_abaqus(matrix, filename='output.xlsx'):

    def generate_labels_notation_python(n):
    def save_matrix_to_excel_notation_python(matrix, filename='output.xlsx'):
