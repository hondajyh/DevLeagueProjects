### SPRINT #3: Math Expression Store & Evaluator
#### SPRINT CONCEPT: Data Operations

12/13/2017

**Author:** Jon Honda


__*Project Description:*__ I need to build a system to store, retrieve, and evaluate math expressions as a part of a larger project. The system will store the expression, expression name, and its variables in a single table.

Special specification for variables:
  * The variables shall be held in a single data field.
  * The variable data field will hold all the attributes about each variable in the expression.
  * The variable's value will be found in another database table and field. So, the variable attributes needs to define the table and field to find the value in.
  * Also, the variables may be expression names. So, one of the variable attributes needs to identify if the variable is a dbValue (See above) or a expression.

Special specification for expressions evaluation:
  * The expression evaluator will accept an expression and return the expression dbValue
  * The expression evaluator will evaluate expression using python's eval() function. Acknowledging this is not generally safe process, but okay for learning how things work.
  * Evaluator will get variable value using one of the two methods below. It will replace variable in expression string w/ the obtained value
    * If evaluator identifies variable as a dbValue, then it will get the value from the database
    * If evaluator identifies variable as a expression, then it will pass the expression into the Evaluator function

* ***********Solution: Make database:******************
import sqlite3
import os
db = sqlite3.connect(':memory:') #### allocate memory
try:
    os.remove('_jonhonda_dat//toyExprDB') #### remove existing db if it exists
    print ('killed existing db')
except FileNotFoundError as err:
    print('db does not exist - okay make it') #### do nothing
db = sqlite3.connect('_jonhonda_dat//toyExprDB')  #### make a database
cursor = db.cursor()
#### build expression table:
cursor.execute ('''CREATE TABLE expressions(
    expression_id INTEGER PRIMARY KEY,
    expression_name TEXT,
    expression_str TEXT,
    expression_data_type TEXT,
    vars BLOB)''')
db.commit()
#### build a toy table that hold variable values:
cursor.execute ('''CREATE TABLE sizes(
    size_id INTEGER PRIMARY KEY,
    area FLOAT,
    volume FLOAT,
    length FLOAT
)''')
db.commit()


* *********SOLUTION: ADD Data to DB: *****************
#### put some values into variable value tables
cursor.execute ('''INSERT INTO sizes(area, volume, length) VALUES(?,?,?)''', (25.6,33.1,2.5))
cursor.execute ('''INSERT INTO sizes(area, volume, length) VALUES(?,?,?)''', (45.6,23.4,0.5))
cursor.execute ('''INSERT INTO sizes(area, volume, length) VALUES(?,?,?)''', (14.8,2.6,1.6))
db.commit()

#### make some expression data:
expression_name = 'SIZE'
expression_str = 'area*3+volume-length*length*length-SIDE'
expression_data_type = 'float'
#### store variable data in a dictionary object
#### format: [dbVarType, dbTable, dbField, dbQryOnUniqueField, valDataType]
VARS = {'area':['db_val','sizes','area', 'size_id', 'FLOAT'],
       'volume':['db_val','sizes','volume', 'size_id', 'FLOAT'],
       'length':['db_val','sizes','length', 'size_id', 'FLOAT'],
        'SIDE':['expression','expressions','expression_str', 'expression_name', 'FLOAT']
       }
#### now serialize the variable dictionary w/ pickling:
import pickle
# Pickle the 'data' dictionary using the highest protocol available.
# use the dumps command to write pickle to a string
mypickle = pickle.dumps(VARS, pickle.HIGHEST_PROTOCOL) ####uses the latest pickling version
#### NOW INSERT DATA TO DB:
cursor.execute ('''INSERT INTO expressions(expression_name, expression_str, expression_data_type, vars) VALUES(?,?,?,?)''',(expression_name,expression_str,expression_data_type,mypickle))
db.commit()

expression_name = 'SIDE'
expression_str = 'length*4'
expression_data_type = 'float'
#### store variable data in a dictionary object
#### format: [dbVarType, dbTable, dbField, dbQryOnUniqueField, valDataType]
VARS = {'length':['db_val','sizes','length', 'size_id', 'FLOAT']}
#### now serialize the variable dictionary w/ pickling:
import pickle
# Pickle the 'data' dictionary using the highest protocol available.
# use the dumps command to write pickle to a string
mypickle = pickle.dumps(VARS, pickle.HIGHEST_PROTOCOL) ####uses the latest pickling version
#### NOW INSERT DATA TO DB:
cursor.execute ('''INSERT INTO expressions(expression_name, expression_str, expression_data_type, vars) VALUES(?,?,?,?)''',(expression_name,expression_str,expression_data_type,mypickle))
db.commit()



* ***************Some code
import pickle
import sqlite3
#### DEFINE GLOBALLY SCOPED CONSTANTS TO HELP UNDERSTAND WHAT ARRAY ELEMENT WE'RE ACCESSING:
####  Constants for expression record query tuple:
C_exprRecTuple_exprName = 0
C_exprRecTuple_exprStr = 1
C_exprRecTuple_exprDataType = 2
C_exprRecTuple_Vars = 3

C_VarDict_VarName = 0 #### Constant for Var Dict Key
#### Constants for Var Dict Array:
C_VarDict_VarType = 0
C_VarDict_StoredTable = 1
C_VarDict_StoredField = 2
C_VarDict_QryOnUniqueField = 3
C_VarDict_DataType = 4


def EvalExpr(exprRecTuple): #### pass in expression record as a tuple
    procstr = exprRecTuple[C_exprRecTuple_exprStr] #get expression string from record tuple
    Vars = pickle.loads(exprRecTuple[C_exprRecTuple_Vars]) #unpickle to Vars variable
    print('proccessing expression: ' + procstr)
    for aVar in Vars.items(): #iterate thru each Var in Vars, replacing procstr's Var instances w/ Var's value
        procstr = procstr.replace(aVar[C_VarDict_VarName],getVal(aVar,1))
        print('proccessing expression: ' + procstr)
    myVal = eval(procstr)
    print ('  eval(' + procstr + ')=' + str(myVal))
    return myVal


def getVal(aVar, QryOnUniqueFieldVal): #retrieve DB value, or call expression evaluation of passed variable (expects tuple of expression Query)   
    print('    attempting to retrieve value for: ', aVar[C_VarDict_VarName] )
    #### handle different db_Var types:

    strdbVal = 'fault_if_still_this'

    if aVar[1][C_VarDict_VarType] == 'db_val': #### value is housed somewhere in database. get value
        dbTableName = aVar[1][C_VarDict_StoredTable]
        dbFieldName = aVar[1][C_VarDict_StoredField]
        dbQryOnUniqueField = aVar[1][C_VarDict_QryOnUniqueField]
        QryStr = '''SELECT ''' + dbFieldName + ''' FROM ''' + dbTableName + ''' WHERE ''' + dbQryOnUniqueField + '''=''' + str(QryOnUniqueFieldVal)
        print('      QUERY:' + QryStr)
        cursor.execute(QryStr)
        dbVal = cursor.fetchone()[0] #### return record as value
        strdbVal = str(dbVal) #### cast to string
        print('       QUERY RESULT: ' + aVar[C_VarDict_VarName] + '=' + strdbVal)

    elif aVar[1][C_VarDict_VarType]=='expression':
        print ('      This is an expression. Prepare to re-enter EvalExpr...')
        dbTableName = 'expressions'
        dbFieldName = 'expression_name'
        QryStr = '''SELECT expression_name, expression_str, expression_data_type, vars \
        FROM ''' + dbTableName + ''' WHERE ''' + dbFieldName + '''= \'''' +  aVar[C_VarDict_VarName] + '''\''''
        print('      QUERY:' + QryStr)
        cursor.execute(QryStr)
        dbVal = cursor.fetchone() #### return record as tuple
        print('       Reentering EvalExpr....')
        strdbVal = str(EvalExpr(dbVal)) #### cast to string

    else:
        print (strdbVal)

    return strdbVal


#### main code here:
db = sqlite3.connect('_jonhonda_dat//toyExprDB')  #### make a database
cursor = db.cursor()
cursor.execute ('''SELECT expression_name, expression_str, expression_data_type, vars FROM expressions WHERE expression_name = 'SIZE' ''')
dbVars = cursor.fetchone() #### return record as tuple
print('RUN EvalExpr on expression: ' + dbVars[C_exprRecTuple_exprStr]  + ' for Size Item 1\n'  )
EvalExpr(dbVars)
db.close()
