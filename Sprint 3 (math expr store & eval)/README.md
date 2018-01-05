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


SEE ExprEval.py for solution.  Description of script:  
* script file will create a database and populate with sample data, including expressions and data values for variables in the expression.
* variables will be stored as serialized data using python pickling process
* script will then run the evaluator code segment which does the following:
  1. get expression information, including expression and variables (depickle.  it is held as serialized data)
  2. for each variable, do the following:
     a. extract variable type information from the retrieved variable information
     b. if variable type is db_var then variable's value is held in a data table in the database. retrieve table name, field name, and unique field to query on.
     c. retrieve value by querying on information for a given unique field (we assume we know what the unique field is ahead of time)
     d. in the expression, replace varible with retrieved value  
     e. if variable type is expression, then recursively reenter expression evaluator.
  3. after all variables have been processed, evaluate expression
