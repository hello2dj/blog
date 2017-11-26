### file manager read whole file 

### viewport display the textual content graphically

### scrolbar need to give the use some way to access different parts of the file

### use cases
  *#### three candidate elements
  *#### behavior of the textbrowser
  *#### how the user will use the intended solution
  * #### move handle
  * #### change window (viewport) size

### analysis model
  * #### UML clas-model diagram
  * #### reactangles for classes
  * #### Each reactangle is divided vertically
  * #### Lines between the components denote relationships
### opeartions
  comprise those actions that the user can undertake to interact with the textbrowser
### percepts
  - handle's position
  - size of viewport
  - text
  - size handle

### FileMananger
  - supplies the contents of the file
  - not visible to the user
  - textbrowser interacts with the file system
  - document is a percept that is supplied the os action

### relationships
  - associations
  - aggregation
  - generalization