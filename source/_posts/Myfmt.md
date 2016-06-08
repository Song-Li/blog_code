title: "Myfmt"
date: 2015-10-05 22:48:09
tags: ADV Programming
---
Author: Song Li
##AIM##
This program is use to do text format things. Users can use this program to limit the width of every line into a particular length and can uniform every blanks.
<!--more-->
##USAGE##
Shown as "--help", there are two major functions:
-u to format the text
-w [length] to limit the width of a line into a
--h to show the help page
--v to show the version info
##COMPILE##
Using

> gcc -o fmt.out fmt.c

##FILE STRUCTURE##

>fmt.c: the main code of this program
>inputjane: the inputfile that I use to test the efficiency
>HW2 fmt readme.pdf: Document of this program

##RESULT##
-w [length]: if the original line is longer than the number length, this line will be cut into several lines. After this operation, the maximum length of every line will be the input number “length” and
every new line which comes from a same original line will share the same indent just as the original line. They have the same number of leading blanks. And the words will not be separated which means if a word is right at the cut point of the original line, this word will be moved to the next line. So no word will be cut. The default length of every new line is 75.
-u: if there are more than one blank characters such as ‘ ’, ”\t”, some of them will be removed. The result is that between every two words there will be only one blank. After every sentence, which means there are some characters like ‘.’, ‘?’, ‘!’ shown up at the end of a sentence, there will be two blanks. After this action, the length of every line will be limited within 75 characters by default. If users input the maximum length of every line, this limitation will be the input number.
##INPUT AND OUTPUT##
The input text must come from an input file. The result will be output as a “stdout” stream. The
input format are exampled here:
>./fmt –u –w 50 inputfile1 inputfile2 > outputfile

##ALGORITHM DESIGN##
###Overall###
This program formats texts line by line. In this way, it can handle a very large file in a very small memory cost. It opens the first file, reads a single line into an array, formats it and output it into a particular file and then the next line. After handling a file, it will handle another in the same way.
###Uniform function###
In this function, there are two strings. The first one is the original string. The second string is the new string. There will be two pointers. The first pointer points at the original string, the second one points at the new string. The first pointer, which points at the original string, always keep moving forward.
1) When the first pointer points at a normal character, this character will be copied to the new string, and the pointer points at the new string will move forward too.
2) When the first pointers points at a blank such as ‘ ’, ‘\t’, this pointer will keep moving until it points at a normal character which is not a blank character. At the same time, the second pointer will add a blank to the second string. If the last no-blank character is ‘.’, ‘?’ or ‘!’, the second pointer will add two blanks to the second string.
After this, the result string will be sent to the –w procession. After the –w procession, we get the final result of this line.
###Width function###
In this function, there also are two strings and two pointers like the uniform function. In order to have a better performance, I designed an algorithm for this function.
1) Reset the length limitation to the original limitation minus the number of leading blanks. In this way, we can easily get the real length of characters.
2) The original string pointer jumps from the start position of the new line to the expected end position, which is the start position add the length limitation.
3)If this position is in the middle of a word, then the first pointer will move backward until the pointer finds a blank character or reaches the new start position.
4)If the pointer gets a blank, it will continue move backward to finds a normal character.
5) If the pointer reaches the new start position, it will move forward to find the end of this word.
6) Using “memcpy” to copy this new line which from the new start position to the end position to the aim string.
7) Set the new start to the position of the end position and repeat the steps from 2 to 7.
###Efficiency###
I compared the run time of my program and the system fmt program. The data to test this two programs is a novel called “Jane Eyre”, which contains 21062 lines. Here is the result. (The time is the average time of 10 tests, using linux time function). I’m sure that the system’s fmt is much better than mine. There are lots of things that I didn’t considered like the proper real length, the dictionary and so on.

![](/image/20151005.jpg)

Here is the code:

```
#include <stdio.h>
#include <stdlib.h>
#include <getopt.h>
#include <string.h>
#define FILE_LEN 10000
static struct option long_options[] = {
    {"help", no_argument, NULL, 'h'},
    {"version", no_argument, NULL, 'v'},
    {"uniform", no_argument, NULL, 'u'},
    {"width", required_argument,NULL, 'w'},
    {NULL, no_argument, NULL ,0}
};
struct Task{
    int w,u,h,v;
    int w_len;
}tasks;
int i;
int startblank = 0;
int isBlank(char c){
    return c == ' ' || c == '\t' || c == '\n';
}
int isFinish(char c){
    return c == '.' || c == '?' || c == '!' ;
}
void showhelp(){// help info
    printf("\nUsage: fmt [-WIDTH] [OPTION]... [FILE]...\n");
    printf("Format each line in the FILE(s)\n\n");
    printf("  -w, --width\t\tmaximum line width (default of 75)\n");
    printf("  -u, --uniform\t\tone space between words, two after sentences\n\n");
    printf("      --help\t\toutput this help and exit\n");
    printf("      --version\t\toutput the version and exit\n\n");
    printf("Report myfmt bugs to sol315@lehigh.edu\n");
    return ;
}
void error(int id,int msg, char *file){//errors
    printf("Error from fmt: ");
    switch (id){
        case 1:
            printf("Input file not found\n");
            break;
        case 2:
            printf("Input file reading error in file \"%s\" line %d.\n",file,msg);
            break;
        case 3:
            printf("The -w function requires the length of the line bigger than 0\n");
            break;
        case 4:
            printf("Fmt requires input arguments\n");
            showhelp();
            break;

    }
    return ;
}

/**
 * this function is used to do the -w work
 * @param 
 *          char *ret: the address of the string that needed to be changed
 *          int len: the length of that string
 * @return  the address of the changed string
 **/
char *runw(char *ret,int len){
    char buffer[FILE_LEN] = {0};    
    int ret_len = 0;//current length of the new string
    int jump = tasks.w_len;
    if(strlen(ret) == 1) return ret;
    strcpy(buffer,ret);//when this function finished, the "buffer" space will gone. So we use buffer to
                       //store the original string, use "ret" to store the new string
    memset(ret,0,FILE_LEN);
    if(jump <= startblank) jump = startblank + 1;//if the max width smaller than startblank, this line
                                                 //should place one word. We set jump=startblack+1 to do that
    int posi = jump - 1,pre = startblank;//pre means the start position of the new line in the original string. posi means the current pointer position in original string
    while(posi < len){//this means the process hasn't finished
        while(posi > pre && !isBlank(buffer[posi])) --posi;//if the max width cuts a word, get the start of the word.
        if(posi == pre) while(!isBlank(buffer[posi])) ++posi;//if the first single word longer than the max width, get the end of the word.
        while(isBlank(buffer[-- posi]));//if the position of the pointer is in blank, move backward and get the end of this line.
        posi ++;
		memset(ret + ret_len, ' ', startblank);//put some blanks in the start of the new line
		ret_len += startblank;//get the new length of the new string
		memcpy(ret + ret_len, buffer + pre, posi - pre);//copy the line from the original string to the new one
		ret_len += posi - pre;
		ret[ret_len ++] = '\n';
        while(isBlank(buffer[posi])) ++posi;//skip the blanks and get the start of next line
        pre = posi;
        posi += jump - startblank - 1;//jump to the next possible cut point
    }
	if(pre != len) {//this part is used to handle the last line of the string
		posi = len - 1;//the position of the pointer is the end of the string
		while(isBlank(buffer[posi])) --posi;//get the last no-blank char
		posi ++;
		if(posi != pre) {
			memset(ret + ret_len, ' ', startblank);
			ret_len += startblank;
			memcpy(ret + ret_len, buffer + pre, posi - pre);
			ret_len += posi - pre;
		}
        ret[ret_len ++] = '\n';
	}
	
    return ret;
}
/**
 * this function is used to do the -u task
 * @param   buffer: the address of file 
 *          len: the length of this file
 * @return  the address of the result string
 **/
char *runu(char* ret,int len){
    char buffer[FILE_LEN];
    int ret_len = 0;
    int flag = 0;
    strcpy(buffer,ret);
    memset(ret, 0, FILE_LEN);//end of the string
	memset(ret,' ',startblank);//put some blanks to the start of the new string
	ret_len = startblank;
    for(i = 0;i < len;){
        if(isFinish(buffer[i])) flag = 1;//if this char means the end of a sentence, set flag true
        if(!isBlank(buffer[i])) ret[ret_len ++] = buffer[i ++];//if it is a normal char, copy it 
        else{//if it is a blank char
            if(i) ret[ret_len ++] = ' ';//if it isn't the start of the string, put a ' ' in it.
            if(flag){
                ret[ret_len ++] = ' ';//if it is the end of a sentence, put another blank in it.
                flag = 0;
            }
            while(isBlank(buffer[++ i]));//skip every blanks. using ++ i instead of i ++ can loop 1 time less
        }
    }
    return runw(ret,ret_len);//every -u needs a -w
}
int getStartBlank(char *buffer,int len){//get the number of start blank
    for(i = 0;i < len;++ i)
        if(!isBlank(buffer[i])) 
            return i;
    return 0;
}
int fmt(char *filename){
    FILE *fp;
    char buffer[FILE_LEN] = {0};
    char *res;
    int linenum = 0;
    res = buffer;
    int len;
    fp = fopen(filename,"r");
    if(NULL == fp){
        error(1,0,NULL);
        return -1;
    }
    while(fgets(buffer,FILE_LEN - 2,fp) != NULL){//format the file line by line
        len = strlen(buffer);
        linenum ++;
        if(ferror(fp)) {
            error(2,linenum,filename);
            return -1;
        }
        startblank = getStartBlank(buffer,len);
        if(tasks.u) res = runu(res,len);//if we do -u, the -w has been done so "if else if"
	    else if(tasks.w) res = runw(res,len);
        printf("%s",res);
    }   
    fclose(fp);
    return 0;
}
void showversion(){
    printf("version 25\n");
    return ;
}
void init(){
    tasks.w = tasks.u = 0;
    tasks.h = tasks.v = 0;
    tasks.w_len = 75;
    return ;
}
int main(int argc,char **argv){
    init();
    int option_index = 0;
    int opt = 0;
    char optstring[4] = "uw:";
    char** infile;
    int file_num = 0;
    while((opt = getopt_long(argc,argv,optstring,long_options,&option_index)) != -1){
        switch (opt){
            case 'u': 
                tasks.u = 1;
                break;
            case 'w':
                if(optarg != NULL) tasks.w_len = atoi(optarg);
                if(tasks.w_len <= 0){
                    error(3,0,NULL);
                    return -1;
                }
                tasks.w = 1;
                break;
            case 'h':
                tasks.h = 1;
                break;
            case 'v':
                tasks.v = 1;
                break;
        }
    }
    if(argc == 1) error(4,0,NULL);
    infile = argv + optind;//get the name of input files
    file_num = argc - optind;//get the number of input files
    for(i = 0;i < file_num;++ i){
        fmt(infile[i]);
    }
    if(tasks.h) showhelp();
    if(tasks.v) showversion();
    return 0;
}
```
