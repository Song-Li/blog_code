title: "Distance Calculator GUI"
date: 2015-10-17 11:38:54
tags: Python
---
Everyone want to calculate the distance between his Goddess and himself. This calculator can meet your need!
This is the first time that I use Python to generate a calculator.
##AIM##
This program is used to get a GUI software for a edit distance calculator
<!--more-->
##USAGE##
- **Use python to run**: Use *python distance.py* to run this program
- **Input**: The input part is located at the top of this GUI. 
    - First is the input of S and T, which is the source string and the target string. The button behind the input box is used to choose a file to get the S and T. **Attention that only the FIRST LINE of the selected file will be read into the input box**
    - Second is the input of step cost of insertion, deletion and substitution a letter. **The range of the input could be adjusted at the menu bar, under** *Option* 
    - The part after that is the check boxes of output. Users can choose to show the full edit matrix, backtrack matrix and the alignment. 
- **Output**: The output is located at the bottom of the GUI. It's the distance, full edit matrix, backtrack matrix and the alignment.
##DESIGN##
The frame work of the design is shown below.
![](/image/20151017design.jpg)
The blue boxes in Figure 1 is the frame I use. In order to align those boxes, I use many frames to get a better effect.
Figure 2 is my initial design
![](/image/20151017hand.jpg)
##FEEDBACK##
- **Yi Hu**: The GUI is neat and user-friendly. People without read me file can use it.
- **XinYang Zhang**: This GUI looks compact and satisfies the requirements.
- **Robert Brotzman - smith**: I think that is good. I would change the names of the strings to someting more meaningful as well as the operation names. Also maybe add names to the buttons to add files.
For Robert's suggestion, I changed my strings to "deletion", "insertion" and "substitution". Which is used in the homework assignment. For the "add file", I changed my "..." to "open". Thank him.
##RUN CAPTURE##
Figure 3-6 is the screenshot of this software.
![Figure 3](/image/201510171.png)
![Figure 4](/image/201510175.png)
![Figure 5](/image/201510173.png)
![Figure 6](/image/201510174.png)

##CODE##
```
from Tkinter import *
from tkFileDialog import *
from tkSimpleDialog import *
from array import *

#this is the main class actually I only have ONE class

class application(Frame):

    #this is the calculation part
    def cal(self):
        #get the value of S and T
        S = self.textS.get("1.0", END)
        T = self.textT.get("1.0", END)
        lens = len(S)
        lent = len(T)
        if lens == 1 and lent == 1: 
            return 
        self.maxlen = 2
        #init the dp and bk with 0 and -
        #bk is the backtrack matrix
        #dp is the full edit matrix
        dp = [([0] * lent) for i in range(lens)]
        bk = [(['-'] * lent) for i in range(lens)]

        #init the first line and first colume
        for i in range(1,lent):
            dp[0][i] = dp[0][i - 1] + self.Sci.get()
            bk[0][i] = '-'
        for i in range(1,lens):
            dp[i][0] = dp[i - 1][0] + self.Scd.get()
            bk[i][0] = '|'

        #use the DP formula to get the full edit matrix and backtrack matrix
        for i in range(1,lens):
            for j in range(1,lent):
                if S[i - 1] == T[j - 1]:
                    dp[i][j] = dp[i - 1][j - 1]
                    bk[i][j] = '\\'
                else: 
                    dp[i][j] = min(dp[i - 1][j] + self.Scd.get(), dp[i][j - 1] + self.Sci.get(), dp[i - 1][j - 1] + self.Scs.get())
                    if dp[i][j] == dp[i - 1][j] + self.Scd.get():
                        bk[i][j] = '|'
                    if dp[i][j] == dp[i - 1][j - 1] + self.Scs.get():
                        bk[i][j] = '\\'
                #here I get the max lenth of all the numbers. To help align the output
                self.maxlen = max(len(str(dp[i][j])),self.maxlen)
        #here between every two numbers I got 2 spaces
        self.maxlen += 2
        #output the backtrack matrix
        self.Dis.delete(first = 0, last = END)
        self.Dis.insert(END, dp[lens - 1][lent - 1])
        if self.backtrack.get() == TRUE:
            self.BM.insert(END, '\n--------------\n')
            self.BM.insert(END, '    ')
            for i in range(0,lent - 1):
                self.BM.insert(END, '  ' + T[i])
            self.BM.insert(END, '\n   0')
            for i in range(0,lens):
                if i != 0:
                    self.BM.insert(END, S[i - 1])
                for j in range(0,lent):
                    if i == 0 and j == 0:
                        continue
                    if i == 0:
                        self.BM.insert(END, '  -')
                    if j == 0:
                        self.BM.insert(END, '  |')
                    if i != 0 and j != 0:
                        self.BM.insert(END, '  ' + bk[i][j])
                self.BM.insert(END, '\n')

        #if the full edit check button is selected, output the full edit matrix
        if self.fulledit.get() == TRUE:
            self.FD.insert(END, '\n--------------\n')
            self.FD.insert(END, ' ' * self.maxlen * 2)
            for i in range(0,lent - 1):
                self.FD.insert(END, T[i].ljust(self.maxlen))
            self.FD.insert(END, '\n' + ' ' * self.maxlen)
            for i in range(0,lens):
                if i != 0:
                    self.FD.insert(END, S[i - 1].ljust(self.maxlen))
                for j in range(0,lent):
                    self.FD.insert(END, str(dp[i][j]).ljust(self.maxlen))
                self.FD.insert(END, '\n')

        #if hte alignment check box is selected, output the alignment
        if self.align.get() == TRUE:
            self.AL.insert(END, '\n--------------\n')
            j = lent - 1
            i = lens - 1
            res = []
            while TRUE:
                #get the backtrack path
                res.append(bk[i][j])
                if bk[i][j] == '\\':
                    i -= 1
                    j -= 1
                elif bk[i][j] == '|':
                    i -= 1
                elif bk[i][j] == '-':
                    j -= 1
                if i == 0 and j == 0:
                    break
            #reverse the backtrack path
            res.reverse()

            i = j = k = 0
            #init the So, way and Ta list. which stores source, way, target information
            So = ['a'] * len(res)
            way = ['a'] * len(res)
            Ta = ['a'] * len(res)
            for p in range(len(res)):
                if res[p] == '\\':
                    So[k] = S[i]
                    Ta[k] = T[j]
                    if S[i] == T[j]:
                        way[k] = '|'
                    else:
                        way[k] = ' '
                    i += 1
                    j += 1
                if res[p] == '|':
                    So[k] = S[i]
                    Ta[k] = '-'
                    way[k] = ' '
                    i += 1
                if res[p] == '-':
                    So[k] = '-'
                    Ta[k] = T[j]
                    way[k] = ' '
                    j += 1
                k += 1
            for i in range(len(res)):
                self.AL.insert(END, ' ' + So[i])
            self.AL.insert(END, '\n')
            for i in range(len(res)):
                self.AL.insert(END, ' ' + way[i])
            self.AL.insert(END, '\n')
            for i in range(len(res)):
                self.AL.insert(END, ' ' + Ta[i])
            self.AL.insert(END, '\n')







    #this function is used to init the class

    def __init__(self, master):
        Frame.__init__(self,master)
        self.Sci = 1
        self.Scd = 1
        self.Scs = 1
        self.maxlen = 2
        self.pack()
        self.create()

    #this function is used to change the range of step cost
    def changeRange(self):
        while(TRUE):
            Range = askinteger("Update the Range","Please input the range")
            if Range > 0:
                break
            if Range == None:
                break
        self.Scd.config(to = Range)
        self.Sci.config(to = Range)
        self.Scs.config(to = Range)

    #this function is used to show the menu bar
    def menu(self):
        menu = Menu()
        self.master.config(menu = menu)
        option = Menu(menu)
        menu.add_cascade(label = "Option", menu = option)
        option.add_command(label = "Range of modify cost", command = self.changeRange)


    #this function is used to get the text boxes which appear at the bottom of the GUI
    def getOut(self, name,father):
        self.frm = LabelFrame(father, text = name, padx = 10, pady = 5)
        lb = Text(self.frm, width = 30, wrap = NONE)
        lb.config(font = ("Courier", 10))
        sl = Scrollbar(self.frm)
        sl.set(0.5,1)
        sl.pack(side = RIGHT, fill = Y)
        lb['yscrollcommand'] = sl.set
        lb.pack(side = LEFT)
        sl['command'] = lb.yview

        sl = Scrollbar(self.frm, orient = HORIZONTAL)
        sl.set(0.5,1)
        sl.pack(fill = X)
        lb['xscrollcommand'] = sl.set
        lb.pack(side = TOP)
        sl['command'] = lb.xview
        self.frm.pack(side = LEFT)
        return lb


    #read S and T from files
    def getFileS(self):
        fd = askopenfilename()
        if fd == "":
            return 
        tmpfile = open(fd,"r")
        self.textS.delete("1.0", END)
        self.textS.insert("1.0", tmpfile.readline()[:-1])
        tmpfile.close()

    def getFileT(self):
        fd = askopenfilename()
        if fd == "":
            return 
        tmpfile = open(fd,"r")
        self.textT.delete("1.0", END)
        self.textT.insert("1.0", tmpfile.readline()[:-1])
        tmpfile.close()

    #main function to show the GUI
    def create(self):
        self.menu()
        Input = LabelFrame(self, text = "Input", padx = 10, pady = 5)

        #input part
        frm_L = LabelFrame(Input, text = "Input S & T", padx = 10, pady = 5)
        
        #the S and T input part
        self.frm_LS = Frame(frm_L)
        Label(self.frm_LS, text = "S", font = ('Arial', 12), padx = 10).pack(side = LEFT)
        self.textS = Text(self.frm_LS,height = 3, width = 18, padx = 10)
        self.textS.pack(side = LEFT)
        Button(self.frm_LS, text = "Open", command = self.getFileS).pack(side = LEFT)
        self.frm_LS.pack(side = TOP)

        frm_LT = Frame(frm_L)
        Label(frm_LT, text = "T", font = ('Arial', 12), padx = 10).pack(side = LEFT)
        self.textT = Text(frm_LT,height = 3, width = 18, padx = 10)
        self.textT.pack(side = LEFT)
        Button(frm_LT, text = "Open", command = self.getFileT).pack(side = LEFT)
        frm_LT.pack(side = TOP)

        frm_L.pack(side = LEFT)

        #the distance input part
        frm_M = LabelFrame(Input, text = "Cost of modify", padx = 10, pady = 5)

        frm_Mcd = Frame(frm_M)
        Label(frm_Mcd, text = "deletion", font = ('Arial', 11), width = 10).pack(side = LEFT)
        self.Scd = Scale(frm_Mcd, orient = HORIZONTAL, width = 10, from_ = 1, to = 5)
        self.Scd.pack(side = LEFT)
        frm_Mcd.pack(side = TOP)


        frm_Mci = Frame(frm_M)
        Label(frm_Mci, text = "insertion", font = ('Arial', 11), width = 10).pack(side = LEFT)
        self.Sci = Scale(frm_Mci, orient = HORIZONTAL, width = 10, from_ = 1, to = 5)
        self.Sci.pack(side = LEFT)
        frm_Mci.pack(side = TOP)


        frm_Mcs = Frame(frm_M)
        Label(frm_Mcs, text = "substitution", font = ('Arial', 11), width = 10).pack(side = LEFT)
        self.Scs = Scale(frm_Mcs, orient = HORIZONTAL, width = 10, from_ = 1, to = 5)
        self.Scs.pack(side = LEFT)
        frm_Mcs.pack(side = TOP)



        #the check box input part
        frm_R = LabelFrame(Input, text = "Pick the output", padx = 10, pady = 5)

        self.fulledit = BooleanVar()
        Checkbutton(frm_R, text = "full edit",height = 2, width = 18, anchor = W, variable = self.fulledit).pack(side = TOP, anchor = W)
        self.backtrack = BooleanVar()
        Checkbutton(frm_R, text = "backtrack matrix", height = 2, width = 18, anchor = W, variable = self.backtrack).pack(side = TOP, anchor = W)
        self.align = BooleanVar()
        Checkbutton(frm_R, text = "alignment", height = 2, width = 18, anchor = W, variable = self.align).pack(side = TOP, anchor = W)

        convert = Frame(self)
        Button(convert, text = "Compare", command = self.cal).pack()


        #this is the output part
        output = LabelFrame(self, text = "Output", padx = 10)
        Disfrm = Frame(output, padx = 10, pady = 5)
        Label(Disfrm, text = "Distance", padx = 10).pack(side = LEFT)
        self.Dis = Entry(Disfrm, width = 10)
        self.Dis.pack(side = LEFT)
        Disfrm.pack(side = TOP)

        #get three textboxes which is the full edit, backtrack matrix and alignment text boxes
        self.FD = self.getOut("full edit", output)
        self.BM = self.getOut("backtrack matrix", output)
        self.AL = self.getOut("alignment", output)

        frm_L.pack(side = LEFT)
        frm_M.pack(side = LEFT)
        frm_R.pack(side = LEFT)

        Input.pack()
        convert.pack()
        output.pack()

if __name__ == "__main__":
    root = Tk()
    root.title("Distance Calculator")
    root.geometry("900x700")
    root.resizable(width = True, height = True)

    app = application(root)
    root.mainloop()
```
