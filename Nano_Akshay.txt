import glob
import xml.etree.ElementTree as ET
import pymysql


class Extracted_data: #Created this class so that the data is encapsulated at one place for easy access
    def __init__(self, Title, PubNum, Abs):
        self.Title = Title
        self.PubNum = PubNum
        self.Abs = Abs
        
filenames = glob.glob("*.xml")  #Regular expression to get all files in local that have the .xml extension
Ext_Data = [];    #Declaring as list

db = pymysql.connect("localhost","root","YOURPASSWORDHERE") #Connection to database - Username and Password needs to be replaced as per the machine needs
cursor = db.cursor()
cursor.execute("create database PatentDB;") #Creating the database on MySQL server
#Creating table in the DB
cursor.execute("create table PatentDB.Nano(Publication_Number varchar(11) primary key, Title varchar(1100), Abstract TEXT);")


for filename in filenames:
    #Code that needs to be executed on each file
    file_dom = ET.parse(filename)
    values = "" #Just to be sure that we don't add duplicate entries
    #Exception Handling - one of the files did not have the node <Abstract>
    if (file_dom.find('Abstract') is None):
        Abs = "Abstract not available"
    else:
        Abs = file_dom.find('Abstract').text.strip()
      
    Data_Obj = Extracted_data(file_dom.find('Title').text.strip(),file_dom.find('PublicationNumber').text.strip(),Abs)
    Data_Obj.Abs = Data_Obj.Abs.replace('"', '\\"')
    Data_Obj.Title = Data_Obj.Title.replace('"', '\\"')
    values = "(\"" + Data_Obj.PubNum + "\",\"" + Data_Obj.Title + "\",\"" + Data_Obj.Abs + "\");"
    Ext_Data.append(Data_Obj) #Although the list is not being used, for simplicity I would prefer keeping all my data at once place 
    sql_query = "insert into PatentDB.Nano values " + values
    #print(sql_query)
    cursor.execute(sql_query) # I did think of bulk commit, but for 1000 records, this was good enough
db.commit()
db.close()