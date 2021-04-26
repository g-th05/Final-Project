import numpy as np
from numpy.linalg import inv
from tinydb import TinyDB,Query

class Model:

    def my_input(self):

        self.m=int(input("Enter the number of buses"))
        self.matadmittance=np.empty((self.m, self.m)) #the admittance matrix without assignment of sign
        self.deltastate=np.empty((self.m-1, 1))       #column matrix w/o the reference nus angle
        self.deltastateerror=np.empty((self.m-1, 1))   #error wala
        self.Fdeltastateerror=np.empty((self.m-1, self.m-1))  #F matrix ma delta ko section
        self.deltaactual=np.empty((self.m, 1))                #column matrix with reference bus ko angle

        self.k=0     # k bhaneko no of power branches in m bus system, for ex- 6 in case of 4 bus system

        for i in range(self.m):
            for j in range(self.m):
                if i<j:
                    self.k+=1 # k patta lako

        self.powerbranch = np.empty((self.k, self.m))  # total power branch ko values in matrix form
        self.matadmittancefinal = np.empty((self.m, self.m))  # Sign halisakepachiko admittance matrix
        self.buspower = np.empty((self.m - 1, 1))  # column matrix of power of each bus
        self.buspowererror = np.empty((self.m - 1, 1))
        self.Fbuspowererror = np.empty((self.m - 1, self.m - 1))
        self.powerbranchactual = np.empty((self.k, 1))  # power branch ko values haru lai single column ma rakheko so that we can concatenate for Y=FX
        self.powerbrancherror = np.empty((self.k, 1))  # error value input from user
        self.Fpowerbrancherror = np.zeros((self.k, self.m - 1))  # total F ma power branch ko section
        self.matadmittancereduced = np.empty(
            (self.m - 1, self.m - 1))  # firat row and first column hatako matrix jasma sign assign gareko cha

        for i in range(self.m):
            for j in range(i + 1, self.m, 1):
                print("enter the admittance of Y", i, j)
                self.matadmittance[i][j] = self.matadmittance[j][i] = float(input()) # only required values of admittance matrai leko ho like only 3 admittances in case of 3 bus system

        for i in range(self.m):
            for j in range(self.m):
                if(i==j):
                    for k in range(self.m):
                        self.matadmittancefinal[i][j]+=self.matadmittance[i][k]
                else:
                    self.matadmittancefinal[i][j]=-self.matadmittance[i][j]
        print(self.matadmittancefinal)


        # print(self.Fbuspowererror)
        # print(self.Fdeltastateerror)
        # print(self.Fpowerbrancherror)

        for i in range(self.m-1):
            print("enter the power of Bus", i+2)
            self.buspower[i][0]=float(input())
        print(self.buspower)



        for i in range(1, self.m):
            for j in range(1, self.m):
                self.matadmittancereduced[i-1][j-1]=self.matadmittancefinal[i][j]
        print("The reduced admittance matrix is:\n", self.matadmittancereduced) #reduced admittance matrix patta lako

        return (self.matadmittancefinal)
        return (self.buspower)
        return(self.matadmittancereduced)

    def calculation(self):
        self.deltastate=np.dot(inv(self.matadmittancereduced), self.buspower)
        print("The state variable values are:\n")
        print(self.deltastate)   #yesle chai (m-1) wata matrai delta ko value nikalcha, not of regference bus

        self.deltaactual[0][0]=0

        for i in range(self.m-1):
            self.deltaactual[i+1][0]=self.deltastate[i][0]
        print("The total angles of all values:")
        print(self.deltaactual) # yesle chai m wata bus ko nai angle nikalcha, including of the reference bus

        for i in range(self.m-1):
            for j in range(i+1, self.m):
                self.powerbranch[i][j]=(-self.matadmittancefinal[i][j])*(self.deltaactual[i][0]-self.deltaactual[j][0]) # Pij nikaleko
        #print(self.powerbranch)

        l=0
        for i in range(self.m):
            for j in range(self.m):
                if i<j:

                    self.powerbranchactual[l][0]=self.powerbranch[i][j] # yo chai m aile power branch matrix lai single column ma lageko
                    l += 1
                # else:
                #     self.powerbranchactual[i][j]=0
        #print("in the column matrix form:")
        #print(self.powerbranchactual)
        #print(type(self.powerbranchactual))
        return(self.deltastate)
        return(self.deltaactual)
        return(self.powerbranch)
        return(self.powerbranchactual)

    def error_input(self):
        print("Enter the errornous value of branch Power:\n")
        for i in range(self.k):
            self.powerbrancherror[i][0]=float(input())
        #print(self.powerbrancherror)
        print("Enter the errornous value of delta\n")
        for i in range(self.m-1):
            self.deltastateerror[i][0]=float(input())
        #print(self.deltastateerror)
        print("Enter the erronous value of Bus Power\n")
        for i in range(self.m-1):
            self.buspowererror[i][0]=float(input())
        #print(self.buspowererror)
        self.Y=(np.concatenate((self.powerbrancherror, self.deltastateerror, self.buspowererror), axis=0))
        print("The required Y matrix as the combination of all is: \n\n")
        print(self.Y)
        return(self.Y)
        return(self.buspowererror)
        return(self.deltastateerror)
        return(self.powerbrancherror)

    def find_Fbranchpower(self):
        for i in range(0,self.k):
            (a,b) = self.indexing(i,self.m)
            #print((a,b))
            if a==0:
                #print(a)
                for t in range(self.m-1):
                    #print(t)
                    if t==(b-1):
                        #print("This is called")
                        self.Fpowerbrancherror[i][t] = -(self.matadmittance[a][b])
            else:
                for t in range(self.m - 1):
                    if t == (a - 1):
                        self.Fpowerbrancherror[i][t] = (self.matadmittance[a][b])
                    if t ==(b-1):
                        self.Fpowerbrancherror[i][t]=-(self.matadmittance[a][b])
        #print(self.Fpowerbrancherror)
        return(self.Fpowerbrancherror)

    def indexing(self,x, m):  # to know the index of each branch power, i.e, 2 means 12 in 3 bus system, yo indexing value herera admittance assign gareko cha
        a = 0
        b = 1

        if x == 0:
            return (a, b)
        for i in range(x):
            if b < m - 1:
                b = b + 1
            else:
                a = a + 1
                b = a + 1
        return (a, b)

    def find_Fbuspower(self):                 #logic in my project copy
        for i in range(self.m-1):
            for j in range(self.m-1):
                if i==j:
                    for k in range(self.m):
                        self.Fbuspowererror[i][j]+=self.matadmittance[i+2-1][k+1-1]
                elif(i<j):
                    self.Fbuspowererror[i][j]=self.Fbuspowererror[j][i]=-(self.matadmittance[i+2-1][j+2-1])
        #print(self.Fbuspowererror)
        return(self.Fbuspowererror)

    def find_Fbusdelta(self):
        for i in range(self.m-1):
            for j in range(self.m-1):
                if i==j:
                    self.Fdeltastateerror[i][j]=1
                else:
                    self.Fdeltastateerror[i][j]=0
        #print(self.Fdeltastateerror)
        return(self.Fdeltastateerror)

    def find_statevariable(self):
        self.F=(np.concatenate((self.Fpowerbrancherror, self.Fdeltastateerror, self.Fbuspowererror), axis=0))
        print("The total F matrix is : \n\n")
        print(self.F)
        print("\n\n")
        return(self.F)

    def run_wo_weight(self):
        Ftranspose = np.transpose(self.F)
        x = np.dot(Ftranspose,self.F)
        x = inv(x)
        x = np.dot(x,Ftranspose)
        x = np.dot(x,self.Y)
        print("The required values of delta are:\n\n")
        print(x)


a=Model()
a.my_input()
a.calculation()
a.error_input()
a.find_Fbranchpower()
a.find_Fbusdelta()
a.find_Fbuspower()
a.find_statevariable()
a.run_wo_weight()






# # ***************************************************************************
#     def my_input(self,str):
#         self.m = int(input("Enter the number of row for " + str + ": "))
#         self.n = int(input("Enter the number of columns for " + str + ": "))
#         Mat1 = np.empty((self.m, self.n))
#         for i in range(self.m):
#             for j in range(self.n):
#                 temp = float(input())
#                 Mat1[i][j] = temp
#
#         print(Mat1)
#         return Mat1
#
# #*****************************************************************************
#
#     def Name(self,str):
#         self.name = str
# # *******************************************************************************
#
#     def get_input(self):
#         self.F = self.my_input("measurement matrix")
#         self.y = self.my_input("raw data")
# # *******************************************************************************
#     def run_wo_weight(self):
#         Ftranspose = np.transpose(self.F)
#         x = np.dot(Ftranspose,self.F)
#         x = inv(x)
#         x = np.dot(x,Ftranspose)
#         x = np.dot(x,self.y)
#         self.result1 = x
#         self.result2 = 0
#
# # *******************************************************************************
#     def run_with_weight(self):
#         Ftranspose = np.transpose(self.F)
#         x = np.dot(Ftranspose,self.w)
#         x = np.dot(x,self.F)
#         x = inv(x)
#         x = np.dot(x,Ftranspose)
#         x = np.dot(x,self.w)
#         x = np.dot(x,self.y)
#         self.result2 = x
#
# # ****************************************************************************
#     def save(self):
#         db = TinyDB('./data/db.json')
#         entry = Query()
#         temp = db.search(entry.name == self.name)
#         if len(temp)==0:
#             db.insert({'name':self.name,'F':self.F.tolist(),'y':self.y.tolist(),'result1':self.result1.tolist(),'result2':self.result2.tolist()})
#         else:
#             db.update({'name':self.name,'F':self.F.tolist(),'y':self.y.tolist(),'result1':self.result1.tolist(),'result2':self.result2.tolist()},entry.name==self.name)
#
# # ****************************************************************************
#     def load(self):
#         db = TinyDB('./data/db.json')
#         entry = Query()
#         temp = db.search(entry.name == self.name)[0]
#         self.F = np.array(temp['F'])
#         self.y = np.array(temp['y'])
#         self.result1 = np.array(temp['result1'])
#         self.result2 = np.array(temp['result2'])
#         self.m = self.F.shape[0]
#         self.n = self.F.shape[1]
# # *********************************************************************************************************************************************
#     def display(self):
#         print("The values are:\nF= \n",self.F,"\ny=\n",self.y,"\nResult without weight=\n",self.result1,"\nResult with weight=\n",self.result2)
# # ****************************************************************************************************************************************************
#     def weight(self):
#         self.w = np.eye(self.m,self.m)
#         for i in range(0,self.m,1):
#             self.w[i][i]=float(input("enter the weight:"))
#         print(self.w)
#
# # ******************************************************************************************************************************************************************
#
# def display_all_values():
#     print('Available Datasets:\n')
#     db = TinyDB('./data/db.json')
#     temp = db.all()
#     for i in temp:
#         print(i['name'])
#     print("***************************************")
#
# flag = int(input("Do you want to enter data or load from database:(1 for loading from database and 2 for manual entry): "))
#
#
# if flag==1:
#     display_all_values()
#     name = str(input("Enter Filenumber"))
#     my_model = Model()
#     my_model.Name(name)
#     my_model.load()
#     my_model.run_wo_weight()
#     #my_model.display()
#     my_model.weight()
#     my_model.run_with_weight()
#     my_model.display()
#     my_model.save()
# else:
#     name = str(input("Enter Filenumber"))
#     my_model = Model()
#     my_model.Name(name)
#     my_model.get_input()
#     my_model.run_wo_weight()
#     #my_model.display()
#     my_model.weight()
#     my_model.run_with_weight()
#     my_model.display()
#     my_model.save()
#
