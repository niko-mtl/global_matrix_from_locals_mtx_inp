
# 16:52 04/18/2024 
# INSA-Toulouse, France
# by Nick Tokmantsev

==============================================================================================
==                                                                                          ==
==                                  Description des matrices                                ==
==                                                                                          ==
==============================================================================================

Abaqus et Python/Pyxel-DIC utilisent des méthodes de disposition des matrices différentes.

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
    C'est un vecteur de déplacement. Chaque élément du vecteur, soit chaque déplacement,
    est écrit A_i, où "A" représente la direction de DDL (U pour l'axe x et V pour l'axe y) et
    "i" est le nœud où le déplacement a lieu.
    
    Exemple: V_420 signifie que c'est le déplacement du nœud 420 dans la direction Y.



Vecteur F:
    C'est un vecteur de force de réaction. Chaque élément du vecteur, soit force résultante, 
    est écrit F_ai, où "a" représente la direction de DDL (u pour l'axe x et v pour l'axe y) et
    "i" est le nœud où ce DDL est appliqué.
    
    Exemple: F_x35 signifie que c'est la force résultante au nœud 35 dans la direction X.



Matrice K:
    C'est une matrice de rigidité. Chaque élément de la matrice est écrit K_ai_bj, où a et b 
    sont les directions de rigidité ("x" pour X et "y" pour Y) et "i"/"j" sont les numéros des
    nœuds.
    
    Exemple: K_x420_y69 signifie la rigidité entre le nœud 420 suivant l'axe X et le nœud 69 
	     suivant l'axe Y (avec des déplacements respectifs).



Les valeurs numériques des éléments ayant le même indice sont identiques. La seule chose qui 
change est la position. 

    Exemple: K_python et K_abaqus sont de dimensions mxm, m>2*15
	     K_x2y15_python == K_x2y15_abaqus, même si 
	     K_x2y15 sur python se trouve dans la position [ 2 ,m/2+15] et
	     K_x2y15 sur abaqus se trouve dans la position [ 4 ,  30  ] 












==============================================================================================
==                                                                                          ==
==                               Description des fonctions                                  ==
==                                                                                          ==
==============================================================================================

===================    1/ read_INP(filename)

Données d'entrée :
    filename - le nom du fichier .INP

Données de sortie: 
    node_mapping - un dictionnaire contenant la numérotation des éléments
    number_dof   - le nombre des DDL du système (nombre des noeuds multiplié par 2)

    Le contenu du node_mappings est noté sous forme suivante (mapping):

                Key Type Size    Value
                 0  list  4   [1, 2, 5, 4]
                 1  list  4   [2, 3, 6, 5]
                 2  list  4   [4, 5, 8, 7]
                 3  list  4   [5, 6, 9, 8]

    Cette bibliothèque contient l'index (key) de chaque élément et l'ordre de connectivité
    des nœuds locaux dans le système global. Par exemple,

                Key Type Size    Value
                 0  list  4   [1, 2, 5, 4]

    Peut être décrit de manière suivante :

            Numéro de l'élément - 0
            Premier   noeud local est au 1-er noeud global
            Second    noeud local est au 2-ème noeud global
            Troisième noeud local est au 5-ème noeud global
            Quatrième noeud local est au 4-ème noeud global

    Les Type et Size ne sont pas importants pour la suite du code. Ils sont toujours "list" et
    4.
    

Fonctionnement:
    Les données contenant la numérotation se trouvent entre les lignes "*Element, type =CAX4" 
    et "*Nset, nset=Set-1, generate". La fonction lit le fichier INP ligne par ligne, prend 
    toutes les valeurs qui se trouvent entre ces lignes et les transforme en une bibliothèque.






===================    2/ read_MTX(filename, t)

Données d'entrée :
    filename - le nom du fichier .MTX
    t        - le nombre de DDL de chaque élément

Données de sortie: une dictionnaire de forme suivante:

                Key      Type         Size     Value
                 0  Array of float64 (8, 8)   [....]
                 1  Array of float64 (8, 8)   [....]
                 2  Array of float64 (8, 8)   [....]
                 3  Array of float64 (8, 8)   [....]

    Cette bibliothèque contient l'index (key) de chaque élément et une matrice de rigidité 
    locale. Les Type et Size ne sont pas importants pour la suite du code. Ils sont toujours
    "Array of float64" et (8,8).
    

Fonctionnement:
    La fonction lit le fichier MTX ligne par ligne en assignant et affectant à chacun des 4 
    éléments variables suivantes (dans l'ordre) :
    
    		            element, row, col, value
    
    Avec ces valeurs, la fonction crée la matrice locale de chaque élément.







===================    3/ assemble_matrice_globale(element_matrices, node_mapping, dof)

Données d'entrée :
    element_matrices - la bibliothèque des matrices locales
    node_mapping     - la bibliothèque contenant la numérotation des éléments
    dof		     - le nombre des DDL du système 

Données de sortie: la matrice globale dans la notation Abaqus (x1,y1, x2,y2...)
    

Fonctionnement:
    On crée une matrice de rigidité vide de dimensions mxm, m = dof

    On remplit cette matrice en faisant appel aux matrices locales et au mapping des éléments.








===================    4/ rearrange_matrix_abacus_to_python(matrix)

Données d'entrée :
    matrix - la matrice globale K en notation Abaqus (x1,y1, x2,y2...)
    
Données de sortie:
    rearranged_matrix - la matrice globale en notation Python (x1,x2...y1,y2...)
    

Fonctionnement:
    La fonction déplace les éléments de la matrice selon la notation Python.





===================    5/ autres fonctions
Les fonctions suivantes sont utilisées pour créer les fichiers excel et une mise en page.

    def generate_labels_notation_abaqus(n)
    def save_matrix_to_excel_notation_abaqus(matrix, filename='output.xlsx'):

    def generate_labels_notation_python(n):
    def save_matrix_to_excel_notation_python(matrix, filename='output.xlsx'):
