title: "Matrix invert"
date: 2015-10-07 20:25:18
tags: ADV Programming
---
Author: Song Li
[Download Project](/download/Matrix-invert.zip)
##AIM##
This program is used to generate, invert matrix. Just use row addups and swaps rows randomly to generate a invertable matrix.
<!--more-->
##USAGE##
- Invert Matrix: To use the invert function, users need to input the Maxtric File right after the program. Here is an example: $$./debug/invert~./debug/input$$
- Generate Matrix: To use the generate function, users need to input $-g$ after the program and the size of matrix after the $-g$. Here is an example:
$$./debug/invert~-g~3$$
This command will generate a $3*3$ matrix, and this maxtrix will be output to the terminal and stored in the $./debug/generate$ file at the same time.
##COMPILE##
Just use $make$ in the root dir of this program to compile this program. I didn't modify the $Makefile$
##FILE STRUCTURE##
- Makefile
- README.pdf
- debug/: contains the runable program and generated matrix
- matrix1-sol315.txt
- matrix2-sol315.txt
- object/
- pivoting-sol315.txt
- source/main.cpp: main code of this program
- source/StdAfx.cpp: useless
- source/StdAfx.h: head file 
- source/parseMatrix.cpp: this code is used to input and output the matrix
- source/parseMatrix.h: head file of paraseMatrix.cpp
##RESULT##
Most of the result will be output directely in the terminal. The generated matrix will be output in both terminal and $/debug/generate$ file.
##DESIGN##
In this part, I mainly talk about how I generate a matrix which is invertable. 
Firstly, I generate a matrix which is random double numbers in the diagonal line, and other numbers are all 0. Like this:
$$
\begin{bmatrix}
1.231 & 0.000 & 0.000 & 0.000 \\\\
0.000 & 1.002 & 0.000 & 0.000 \\\\
0.000 & 0.000 & 1923.412 & 0.000 \\\\
0.000 & 0.000 & 0.000 & 0.023 \\\\
\end{bmatrix}
$$

After that, I randomly pick a line, use a random double number times every number of this line, and add it to another random line.If I pick line 2 and number 2, and add this line to the 4th line. The result will be like
$$
\begin{bmatrix}
1.231&0.000&0.000&0.000 \\\\
0.000&1.002&0.000&0.000 \\\\
0.000&0.000&1923.412&0.000 \\\\
0.000&2.004&0.000&0.023 \\\\
\end{bmatrix}
$$
Some times later, we can get a random matrix. Generately speaking, we can get every matrix in this way.

Main function code is here. The whole project can be downloaded from the link at the begining of this blog.

```

////////////////////////////////////////////////////////////////////////////////////////////////////////////////
//Implementation for the Main method
////////////////////////////////////////////////////////////////////////////////////////////////////////////////

#include "StdAfx.h"


/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

inline double getRandDouble(){
    return (double)rand() / (double)rand() * (double)((rand() & 1 << 1) - 1);
}
void defaultOutput()
{
		printf("\n");
		printf("#########################################################\n");
		printf("## 411 Homework: Numerical Analysis                    ##\n");
		printf("## Brian Chen, Oct 20, 2012                            ##\n");
		printf("#########################################################\n");
		printf("\n");
		printf("Usage:\n");
		printf("\n");
		printf("invert \n");
		printf("With no arguments, invert returns this usage statement.\n");
		printf("\n");
		printf("invert <filename>\n");
		printf("Prints out two matrix inverses for the provided matrix.\n");
		printf(" -- the first inverse is computed with simple inversion.\n");
		printf(" -- the second is computed with partial pivoting\n");
		printf("\n");
		printf("invert -format\n");
		printf("Describes the file format for the input file.\n");
		printf("\n");
		printf("#########################################################\n");
		printf("\n");
		exit(0);
}

void fileFormat()
{
		printf("\n");
		printf("#########################################################\n");
		printf("## 411 Homework: Numerical Analysis                    ##\n");
		printf("## Brian Chen, Oct 20, 2012                            ##\n");
		printf("#########################################################\n");
		printf("\n");
		printf("File format for a 3rd order matrix (input):\n");
		printf(" - numbers below are space deliminated.  no tabs.\n");
		printf(" - There are three lines, each with 3 numbers.\n");
		printf(" - They will be parsed as doubles.\n");
		printf(" - Comment lines start with #.  Comment lines cannot.\n");
		printf("   interrupt the 3 lines with the matrix, but can come.\n");
		printf("   before or after.\n");
		printf("\n");
		printf("---- Matrix file below, not including this line ----\n");
		printf("# comment\n");
		printf("# comment\n");
		printf("1.2 3.3 4.4\n");
		printf("1.112 2.24 0\n");
		printf("3.4 2.2 98.23452\n");
		printf("#\n");
		printf("#\n");
		printf("---- Matrix file above, not including this line ----\n");
		printf("\n");
		printf("#########################################################\n");
		printf("\n");
		exit(0);
}


/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

//Inside the result, the result is to be stored as:
//i=0 i=1 i=2
//[0] [3] [6]  //j=0
//[1] [4] [7]  //j=1
//[2] [5] [8]  //j=2
//
//where the above 3x3 matrix is as you would write it on paper.
//


/**
 * this function is mainly used for debug. we can use it to output
 *
 * @param m: the matrix that needed to be output
 *        size: the size of matrix
 * @return NULL
 **/
void output(double  ** m,int size){
    if(DEBUG == 0) return ;//Debug is defined in StdAfx.h 
    for(int i = 0;i < size;++ i){
        for(int j = 0;j < size + size;++ j){
            if(j == size) printf("|\t");
            printf("%f\t",m[i][j]);
        }
        printf("\n");
    }
    printf("\n");

}

/**
 * this function is mainly used to swap two lines in a matrix
 *
 * @param matrix: the matrix that needed to be swaped
 *        a: the first line which should be swaped
 *        b: the second line which should be swaped
 *        size: the size of the matrix
 * @return NULL
 **/

void swap(double ** matrix,int a,int b,int size){
    output(matrix,size);
    double tmp[1000];
    memcpy(tmp,matrix[a],size * 2 * sizeof(double));
    memcpy(matrix[a],matrix[b], size * 2 * sizeof(double));
    memcpy(matrix[b],tmp,size * 2 * sizeof(double));
    output(matrix,size);
    return ;
}


/**
 * this function is used to get the Max line of the matrix and swap it to the first line
 *
 * @param tmp: the matrix
 *        k: the col that we should find the max line
 *        size: the size of matrix
 * @return NULL
 **/
void getMax(double ** tmp, int k,int size){
    int Max = k;
    for(int i = k + 1;i < size;++ i){
        if(fabs(tmp[i][k]) > fabs(tmp[Max][k])) Max = i; //get the max line
    }
    if(Max != k) {
        swap(tmp,Max,k,size);
    }
    return ;
}

/**
 * this function is used to do the most important things, invert the matrix
 *
 * @param matrix: the matrix that needed to be inverted
 *        type: it is the simple one or pivot one
 *              1: pivot 2:simple
 *        size: the size of matrix;
 **/ 
double ** runTmp(double ** matrix, bool type,int size){
	int i = 0;
    double div;

    double ** tmp = new double*[size];

    for(i = 0;i < size;i ++){
        tmp[i] = new double[size + size];
    }


    for(i = 0;i < size;++ i){//read matrix into tmp matrix and add i after matrix
        for(int j = 0;j < size + size;++ j){
            if(j < size) tmp[i][j] = matrix[i][j];
            else tmp[i][j] = (j - size == i);//if it is not the original matrix, add 1s and 0s to the i matrix
        }
    }
    for(int k = 0;k < size;++ k){
        if(type) getMax(tmp,k,size);//if it is pivot one, just get the max line and swap it to the first line
        while(tmp[k][k] == 0 && k < size - 1) swap(tmp,k,k + 1,size), ++k;//try to get the no-zero line 
        if(!tmp[k][k]){// this maxtrix is not invertable
            printf("This Matrix can\'t be inverted\n");
            exit(0);
        }
        for(i = k + 1;i < size;++ i){
            if((div = tmp[i][k] / tmp[k][k]) == 0) continue;
            for(int j = k;j < size + size;++ j){
                tmp[i][j] -= div * tmp[k][j];// solve this line
            }
        }
    }
    for(int k = size - 1;k >= 0;-- k){
        for(i = k;i >= 0;-- i){
            if(!tmp[k][k]){
                printf("This Matrix can\'t be inverted\n");
                exit(0);
            }
            if((div = tmp[i][k] / tmp[k][k]) == 0) continue;
            for(int j = size + size - 1;j >= 0;-- j){
                if(i == k) tmp[i][j] /= tmp[k][k];
                else tmp[i][j] -= div * tmp[k][j]; 
            }
        }
        output(tmp,size);
    }
    return tmp;
}

/**
 * this function is used to do simple inversion
 *
 * @param matrix: the matrix that needed to be inverted
 *        size: the size of matrix
 * @return the matrix that has been inverted
 **/
double ** simpleMatrixInversion( double ** matrix ,int size){
    int i = 0;

    //insert your code here.//
    double ** tmp = runTmp(matrix, 0,size);
    //declare and allocate the result
    double ** result = new double*[size];
    for(i = 0; i<size; i++){
        result[i] = new double[size];
        for(int j = 0;j < size;++ j){
            result[i][j] = tmp[i][j + size];
        }
    }
	return result;
}





/**
 * this function is used to do pivot inversion
 *
 * @param matrix: the matrix that needed to be inverted
 *        size: the size of matrix
 * @return the matrix that has been inverted
 **/

double ** partialPivotInversion( double ** matrix,int size ){
	int i = 0;

	//insert your code here.//
    double ** tmp = runTmp(matrix,1,size);

	//declare and allocate the result
    double ** result = new double*[size];
    for(i = 0; i<size; i++){
        result[i] = new double[size];
        for(int j = 0;j < size;++ j){
            result[i][j] = tmp[i][j + size];
        }
    }
	
	return result;
}


/**
 * this function is used to plus one line to another line
 * this function is used in generate step
 *
 * @param m:the matrix that needed to be changed
 *        size: the size of maxtrix
 * @return NULL
 **/
void linePlus(double ** m, int size){
    double times = getRandDouble();
    int line1 = rand() % size, line2 = rand() % size;
    for(int i = 0;i < size;++ i){
        m[line1][i] += m[line2][i] * times;
    }
}


/**
 * this function is used to generate matrix
 * @param size: the size of matrix that needed to be generated
 * @return the generated matrix
 **/
double ** generateMatrix(int size){
    srand((unsigned)time(NULL));
    double ** m = new double *[size];
    for(int i = 0;i < size;++ i) m[i] = new double[size];
    for(int i = 0;i < size;++ i){
        m[i][i] = getRandDouble();//get the initiate matrix, whose main line are random doubles and others are 0s.
    }
    int times = size * 8; //we need to modify this matrix "times" times. we set times = size * 8
    printf("Time = %d\n",times);
    for(int i = 0;i < times;++ i){
        linePlus(m, size);//add a random line to another random line
    }
    ofstream out("./debug/generate");//output the matrix to a file
    for(int i = 0;i < size;++ i){
        for(int j = 0;j < size;++ j){
            out << m[i][j] << ' ';
        }
        out << endl;
    }
    printf("Matrix has been stored to ./debug/generate\n");
    return m;
}

/**
 * this function is used to compare two matrix
 * @param two matrix and their size
 * @return if they are same
 * tip: this eps is defined in StdAfx.h 
 **/
bool compare(double ** m1, double ** m2,int size){
    for(int i = 0;i < size;++ i){
        for(int j = 0;j < size;++ j){
            if(fabs(m1[i][j] - m2[i][j]) > eps) return false;
        }
    }
    return true;
}





/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

// ###     ###     ###    ####### ###     ###    ###     ### ####### ####### ###   ###   ######   ######  
// ####   ####    #####   ####### ####    ###    ####   #### ####### ####### ###   ###  ########  ####### 
// ##### #####    #####     ###   #####   ###    ##### ##### ###       ###   ###   ### ########## ########
// ###########   ### ###    ###   ######  ###    ########### #######   ###   ######### ####  #### ###  ###
// ### ### ###   ### ###    ###   ### ### ###    ### ### ### #######   ###   ######### ###    ### ###  ###
// ###  #  ###  ###   ###   ###   ###  ######    ###  #  ### ###       ###   ######### ####  #### ###  ###
// ###     ###  #########   ###   ###   #####    ###     ### ###       ###   ###   ### ########## ########
// ###     ###  ######### ####### ###    ####    ###     ### #######   ###   ###   ###  ########  ######## 
// ###     ###  ###   ### ####### ###     ###    ###     ### #######   ###   ###   ###   ######   ######  

//execution format : [executable name] [argument]
int main(int argc, char* argv[])
{
    int size;
	///usage handling
	if( argc == 1 ){
		defaultOutput();
	}

	///file format handling
	if( argc == 2 && ( strcmp(argv[1], "-format")==0 ) ){
		fileFormat();
	}

	///invert
	if( argc == 2 && ( strcmp(argv[1], "-format")!=0 ) ){
		double ** matrix = parseMatrixFile( argv[1], &size );

        if(strcmp(argv[1],"-g") == 0) {
            matrix = generateMatrix(3);
            printf("Generate matrix with order 3\n");
        }
		
		//compute the simple inverse and print it
		double ** simpleInverse = simpleMatrixInversion( matrix,size);
        printf("The simple inverse result is:\n");
		printMatrix(simpleInverse,size);
		
		//compute the inverse with partial pivoting and print it
		double ** partialInverse = partialPivotInversion( matrix,size);
        printf("The pivot inverse result is:\n");
		printMatrix(partialInverse,size);
        if(compare(simpleInverse,partialInverse,size)) printf("Same\n");
        else printf("Different\n");
		
		//clean up
		for(int i = 0; i<size; i++){
			delete[](simpleInverse[i]);
			delete[](partialInverse[i]);
			delete[](matrix[i]);
		}
		delete[](simpleInverse);
		delete[](partialInverse);
		delete[](matrix);
		
	}
    if(argc == 3 && (strcmp(argv[1], "-g") == 0)){
        size = atoi(argv[2]);
        if(size < 2){
            printf("The size of matrix must bigger than 1\n");
            exit(0);
        }
        double ** matrix = generateMatrix(size);
        printf("Matrix size %dx%d generated\n",size,size);
        printMatrix(matrix,size);
    }


	return 0;
}



////////////////////////////////////////////////////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////////////////////////////////////////////////////


```
