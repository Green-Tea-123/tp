---
layout: page
title: Developer Guide
---
* Table of Contents
{:toc}

--------------------------------------------------------------------------------------------------------------------

## **Acknowledgements**

* This application was forked from [SE-EDU's AB3](https://github.com/nus-cs2103-AY2425S1/tp)

--------------------------------------------------------------------------------------------------------------------

## **Setting up, getting started**

Refer to the guide [_Setting up and getting started_](SettingUp.md).

--------------------------------------------------------------------------------------------------------------------

## **Design**

<div markdown="span" class="alert alert-primary">

:bulb: **Tip:** The `.puml` files used to create diagrams in this document `docs/diagrams` folder. Refer to the [_PlantUML Tutorial_ at se-edu/guides](https://se-education.org/guides/tutorials/plantUml.html) to learn how to create and edit diagrams.
</div>

### Architecture

<img src="images/ArchitectureDiagram.png" width="280" />

The ***Architecture Diagram*** given above explains the high-level design of the App.

Given below is a quick overview of main components and how they interact with each other.

**Main components of the architecture**

**`Main`** (consisting of classes [`Main`](https://github.com/AY2425S1-CS2103-F13-4/tp/blob/master/src/main/java/seedu/address/Main.java) and [`MainApp`](https://github.com/AY2425S1-CS2103-F13-4/tp/blob/master/src/main/java/seedu/address/MainApp.java)) is in charge of the app launch and shut down.
* At app launch, it initializes the other components in the correct sequence, and connects them up with each other.
* At shut down, it shuts down the other components and invokes cleanup methods where necessary.

The bulk of the app's work is done by the following four components:

* [**`UI`**](#ui-component): The UI of the App.
* [**`Logic`**](#logic-component): The command executor.
* [**`Model`**](#model-component): Holds the data of the App in memory.
* [**`Storage`**](#storage-component): Reads data from, and writes data to, the hard disk.

[**`Commons`**](#common-classes) represents a collection of classes used by multiple other components.

**How the architecture components interact with each other**

The *Sequence Diagram* below shows how the components interact with each other for the scenario where the user issues the command `delete contact 1`.

<img src="images/ArchitectureSequenceDiagram.png" width="574" />

Each of the four main components (also shown in the diagram above),

* defines its *API* in an `interface` with the same name as the Component.
* implements its functionality using a concrete `{Component Name}Manager` class (which follows the corresponding API `interface` mentioned in the previous point.

For example, the `Logic` component defines its API in the `Logic.java` interface and implements its functionality using the `LogicManager.java` class which follows the `Logic` interface. Other components interact with a given component through its interface rather than the concrete class (reason: to prevent outside component's being coupled to the implementation of a component), as illustrated in the (partial) class diagram below.

<img src="images/ComponentManagers.png" width="300" />

The sections below give more details of each component.

### UI component

The **API** of this component is specified in [`Ui.java`](https://github.com/AY2425S1-CS2103-F13-4/tp/blob/master/src/main/java/seedu/address/ui/Ui.java)

![Structure of the UI Component](images/UiClassDiagram.png)

The UI consists of a `MainWindow` that is made up of parts e.g.`CommandBox`, `ResultDisplay`, `PersonListPanel`, `JobListPanel`, `CompanyListPanel`, `StatusBarFooter` etc. All these, including the `MainWindow`, inherit from the abstract `UiPart` class which captures the commonalities between classes that represent parts of the visible GUI.

The `UI` component uses the JavaFx UI framework. The layout of these UI parts are defined in matching `.fxml` files that are in the `src/main/resources/view` folder. For example, the layout of the [`MainWindow`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/ui/MainWindow.java) is specified in [`MainWindow.fxml`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/resources/view/MainWindow.fxml)

The `UI` component,

* executes user commands using the `Logic` component.
* listens for changes to `Model` data so that the UI can be updated with the modified data.
* keeps a reference to the `Logic` component, because the `UI` relies on the `Logic` to execute commands.
* depends on some classes in the `Model` component, as it displays `Person`, `Job` and `Company` objects residing in the `Model`.

### Logic component

**API** : [`Logic.java`](https://github.com/AY2425S1-CS2103-F13-4/tp/blob/master/src/main/java/seedu/address/logic/Logic.java)

Here's a (partial) class diagram of the `Logic` component:

<img src="images/LogicClassDiagram.png" width="550"/>

The sequence diagram below illustrates the interactions within the `Logic` component, taking `execute("delete contact 1")` API call as an example.

<div markdown="span" class="alert alert-info">:information_source: **Note:** In the diagram below, we see that the command `delete contact 1` causes the function `deletePerson(p)` to be invoked. In fact, `Person` is our class implementation for contacts. For the rest of this document, please note that "person" and "contact" are used interchangeably to refer to the same thing.
</div>

![Interactions Inside the Logic Component for the `delete contact 1` Command](images/DeleteSequenceDiagram.png)

<div markdown="span" class="alert alert-info">:information_source: **Note:** The lifeline for `DeleteCommandParser` should end at the destroy marker (X) but due to a limitation of PlantUML, the lifeline continues till the end of diagram.
</div>

How the `Logic` component works:

1. When `Logic` is called upon to execute a command, it is passed to an `AddressBookParser` object which in turn creates a parser that matches the command (e.g., `DeleteCommandParser`) and uses it to parse the command.
1. This results in a `Command` object (more precisely, an object of one of its subclasses e.g., `DeleteCommand`) which is executed by the `LogicManager`.
1. The command can communicate with the `Model` when it is executed (e.g. to delete a person).<br>
   Note that although this is shown as a single step in the diagram above (for simplicity), in the code it can take several interactions (between the command object and the `Model`) to achieve.
1. The result of the command execution is encapsulated as a `CommandResult` object which is returned back from `Logic`.

Here are the other classes in `Logic` (omitted from the class diagram above) that are used for parsing a user command:

<img src="images/ParserClasses.png" width="600"/>

How the parsing works:
* When called upon to parse a user command, the `AddressBookParser` class creates an `XYZCommandParser` (`XYZ` is a placeholder for the specific command name e.g., `AddCommandParser`) which uses the other classes shown above to parse the user command and create a `XYZCommand` object (e.g., `AddCommand`) which the `AddressBookParser` returns back as a `Command` object.
* All `XYZCommandParser` classes (e.g., `AddCommandParser`, `DeleteCommandParser`, ...) inherit from the `Parser` interface so that they can be treated similarly where possible e.g, during testing.

### Model component
**API** : [`Model.java`](https://github.com/AY2425S1-CS2103-F13-4/tp/blob/master/src/main/java/seedu/address/model/Model.java)

<img src="images/ModelClassDiagram.png" width="800" />


The `Model` component,

* stores the address book data i.e., all `Person`, `Job`, and `Company` objects 
(which are contained in a `UniquePersonList`, `UniqueJobList`, and `UniqueCompanyList` object respectively).
* stores the currently 'selected' `Person`, `Job`, and `Company` objects (e.g., results of a search query) as a 
separate _filtered_ list which is exposed to outsiders as an unmodifiable `ObservableList<Person>`, 
`ObservableList<Job>`, and `ObservableList<Company>` that can be 'observed' e.g. the UI can be bound to this list 
so that the UI automatically updates when the data in the list change.
* stores a `UserPref` object that represents the user’s preferences. This is exposed to the outside as a `ReadOnlyUserPref` objects.
* does not depend on any of the other three components (as the `Model` represents data entities of the domain, 
they should make sense on their own without depending on other components)

[//]: # (Not sure what to do with the diagram and text below)
<div markdown="span" class="alert alert-info">:information_source: **Note:** An alternative (arguably, a more OOP) model is given below. It has a `Skill` list in the `AddressBook`, which `Person` references. This allows `AddressBook` to only require one `Skill` object per unique skill, instead of each `Person` needing their own `Skill` objects.<br>

<img src="images/BetterModelClassDiagram.png" width="450" />

</div>


### Storage component

**API** : [`Storage.java`](https://github.com/AY2425S1-CS2103-F13-4/tp/blob/master/src/main/java/seedu/address/storage/Storage.java)

<img src="images/StorageClassDiagram.png" width="600" />

The `Storage` component,
* can save both address book data and user preference data in JSON format, and read them back into corresponding objects.
* inherits from both `AddressBookStorage` and `UserPrefStorage`, which means it can be treated as either one (if only the functionality of only one is needed).
* depends on some classes in the `Model` component (because the `Storage` component's job is to save/retrieve objects that belong to the `Model`)

### Common classes

Classes used by multiple components are in the `seedu.address.commons` package.

--------------------------------------------------------------------------------------------------------------------

## **Implementation**

This section describes some noteworthy details on how certain features are implemented.

### \[Proposed\] Undo/redo feature

#### Proposed Implementation

The proposed undo/redo mechanism is facilitated by `VersionedAddressBook`. It extends `AddressBook` with an undo/redo history, stored internally as an `addressBookStateList` and `currentStatePointer`. Additionally, it implements the following operations:

* `VersionedAddressBook#commit()` — Saves the current address book state in its history.
* `VersionedAddressBook#undo()` — Restores the previous address book state from its history.
* `VersionedAddressBook#redo()` — Restores a previously undone address book state from its history.

These operations are exposed in the `Model` interface as `Model#commitAddressBook()`, `Model#undoAddressBook()` and `Model#redoAddressBook()` respectively.

Given below is an example usage scenario and how the undo/redo mechanism behaves at each step.

Step 1. The user launches the application for the first time. The `VersionedAddressBook` will be initialized with the initial address book state, and the `currentStatePointer` pointing to that single address book state.

![UndoRedoState0](images/UndoRedoState0.png)

Step 2. The user executes `delete contact 5` command to delete the 5th person in the address book. The `delete` command calls `Model#commitAddressBook()`, causing the modified state of the address book after the `delete contact 5` command executes to be saved in the `addressBookStateList`, and the `currentStatePointer` is shifted to the newly inserted address book state.

![UndoRedoState1](images/UndoRedoState1.png)

Step 3. The user executes `add contact n/David …​` to add a new person. The `add contact` command also calls `Model#commitAddressBook()`, causing another modified address book state to be saved into the `addressBookStateList`.

![UndoRedoState2](images/UndoRedoState2.png)

<div markdown="span" class="alert alert-info">:information_source: **Note:** If a command fails its execution, it will not call `Model#commitAddressBook()`, so the address book state will not be saved into the `addressBookStateList`.

</div>

Step 4. The user now decides that adding the person was a mistake, and decides to undo that action by executing the `undo` command. The `undo` command will call `Model#undoAddressBook()`, which will shift the `currentStatePointer` once to the left, pointing it to the previous address book state, and restores the address book to that state.

![UndoRedoState3](images/UndoRedoState3.png)

<div markdown="span" class="alert alert-info">:information_source: **Note:** If the `currentStatePointer` is at index 0, pointing to the initial AddressBook state, then there are no previous AddressBook states to restore. The `undo` command uses `Model#canUndoAddressBook()` to check if this is the case. If so, it will return an error to the user rather
than attempting to perform the undo.

</div>

The following sequence diagram shows how an undo operation goes through the `Logic` component:

![UndoSequenceDiagram](images/UndoSequenceDiagram-Logic.png)

<div markdown="span" class="alert alert-info">:information_source: **Note:** The lifeline for `UndoCommand` should end at the destroy marker (X) but due to a limitation of PlantUML, the lifeline reaches the end of diagram.

</div>

Similarly, how an undo operation goes through the `Model` component is shown below:

![UndoSequenceDiagram](images/UndoSequenceDiagram-Model.png)

The `redo` command does the opposite — it calls `Model#redoAddressBook()`, which shifts the `currentStatePointer` once to the right, pointing to the previously undone state, and restores the address book to that state.

<div markdown="span" class="alert alert-info">:information_source: **Note:** If the `currentStatePointer` is at index `addressBookStateList.size() - 1`, pointing to the latest address book state, then there are no undone AddressBook states to restore. The `redo` command uses `Model#canRedoAddressBook()` to check if this is the case. If so, it will return an error to the user rather than attempting to perform the redo.

</div>

Step 5. The user then decides to execute the command `list all`. Commands that do not modify the address book, such as `list all`, will usually not call `Model#commitAddressBook()`, `Model#undoAddressBook()` or `Model#redoAddressBook()`. Thus, the `addressBookStateList` remains unchanged.

![UndoRedoState4](images/UndoRedoState4.png)

Step 6. The user executes `clear`, which calls `Model#commitAddressBook()`. Since the `currentStatePointer` is not pointing at the end of the `addressBookStateList`, all address book states after the `currentStatePointer` will be purged. Reason: It no longer makes sense to redo the `add n/David …​` command. This is the behavior that most modern desktop applications follow.

![UndoRedoState5](images/UndoRedoState5.png)

The following activity diagram summarizes what happens when a user executes a new command:

<img src="images/CommitActivityDiagram.png" width="250" />

#### Design considerations:

**Aspect: How undo & redo executes:**

* **Alternative 1 (current choice):** Saves the entire address book.
  * Pros: Easy to implement.
  * Cons: May have performance issues in terms of memory usage.

* **Alternative 2:** Individual command knows how to undo/redo by
  itself.
  * Pros: Will use less memory (e.g. for `delete`, just save the person being deleted).
  * Cons: We must ensure that the implementation of each individual command are correct.

--------------------------------------------------------------------------------------------------------------------

## **Documentation, logging, testing, configuration, dev-ops**

* [Documentation guide](Documentation.md)
* [Testing guide](Testing.md)
* [Logging guide](Logging.md)
* [Configuration guide](Configuration.md)
* [DevOps guide](DevOps.md)

--------------------------------------------------------------------------------------------------------------------

## **Appendix: Requirements**

### Product scope

**Target user profile**:

* solo, freelance third party recruiters
* manages a significant number of candidate contacts
* has to quickly locate contacts that fit a job listing
* prefer desktop apps over other types
* prefers typing to mouse interactions
* is reasonably comfortable using CLI apps
* only deals with English for work

**Value proposition**: manage contacts faster than a typical mouse/GUI driven app with functionality for quick matching of candidates to jobs


### User stories

Priorities: High (must have) - `* * *`, Medium (nice to have) - `* *`, Low (unlikely to have) - `*`

| Priority | As a …​ | I want to …​                         | So that I can…​                                      |
|----------|---------|--------------------------------------|------------------------------------------------------|
| `* * *`  | user    | list all contacts                    |                                                      |
| `* * *`  | user    | create a contact                     |                                                      |
| `* * *`  | user    | remove a contact                     | remove contact information that I no longer need     |
| `* * *`  | user    | list all job listings                |
| `* * *`  | user    | create a job listing                 |  
| `* * *`  | user    | remove a job listing                 | remove job listing information that I no longer need |
| `* * *`  | user    | screen contacts based on job listing | focus on contacting talents that fits the job        |
| `* * *`  | user    | list all companies                   |                                                      |
| `* * *`  | user    | create a company                     | group and organize contacts and job listings         |
| `* * *`  | user    | delete a company                     | remove company information that I no longer need     |
| `* * *`  | user    | view company details                 | view relevant information of a company together      |


### Use cases

(For all use cases below, the **System** is the `TalentConnect` 
and the **Actor** is the `user`, unless specified otherwise)

**Use case: Delete contact**

**MSS**

1.  User requests to list contacts
2.  TalentConnect shows a list of contacts
3.  User requests to delete a specific contact in the list
4.  TalentConnect deletes the contact
5.  TalentConnect returns an updated list of contacts along with a success message.

    Use case ends.

**Extensions**

* 2a. The contact list is empty.

  Use case ends.

* 3a. The given index is invalid.

    * 3a1. TalentConnect shows an error message.

      Use case resumes at step 2.

**Use case: Delete job**

**MSS**

1. User requests to list jobs
2. TalentConnect shows a list of jobs
3. User requests to delete a specific job in the list
4. TalentConnect deletes the job and unmatches all contacts matched to the job
5. TalentConnect returns an updated list of jobs and contacts along with a success message

   Use case ends.

**Extensions**

* 2a. The job list is empty.

  Use case ends.

* 3a. The given index is invalid.
     * 3a1. TalentConnect shows an error message.

       Use case resumes at step 2

**Use case: Delete company**

**MSS**

1. User requests to list companies
2. TalentConnect shows a list of companies
3. User requests to delete a specific company in the list
4. TalentConnect deletes the company, deletes all jobs associated with the company, and unmatches all contacts matched with the aforementioned jobs.
5. TalentConnect returns an updated list of companies, jobs, and contacts along with a success message

   Use case ends.

**Extensions**

* 2a. The company list is empty.

  Use case ends.

* 3a. The given index is invalid.
    * 3a1. TalentConnect shows an error message.

      Use case resumes at step 2

**Use case: Screen a job listing**

**MSS**

1.  User requests to list job listings
2.  TalentConnect shows a list of job listings
3.  User requests to screen all contacts based on the name of a job listing
4.  TalentConnect returns a list of contacts that fits the job's name

    Use case ends.

**Extensions**

* 2a. The list is empty.

  Use case ends.

* 3a. The given index is invalid.

    * 3a1. TalentConnect shows an error message.

      Use case resumes at step 2.

* 4a. The list is empty.

  Use case ends.

**Use case: Add contact** 

**MSS**

1. User requests to add a contact.
2. TalentConnect returns an updated list of contacts with a success message.
   Use case ends.

**Extensions** 

* 1a. The given parameters are invalid. 

    * 1a1. TalentConnect shows an error message.
  
      Use case ends. 

**Use case: Add job**

**MSS**

1. User requests to add a job.
2. TalentConnect returns an updated list of jobs with a success message.
   Use case ends.

**Extensions**

* 1a. The given parameters are invalid.

    * 1a1. TalentConnect shows an error message.

      Use case ends.

**Use case: Add a company**

**MSS**

1. User requests to add a company.
2. TalentConnect returns an updated list of companies with a success message.

   Use case ends.

**Extensions**

* 1a. The given parameters are invalid.
    * 1a1. TalentConnect shows an error message.

      Use case ends.

**Use case: List companies**

**MSS**

1. User requests to list companies.
2. TalentConnect shows the list of companies.

   Use case ends.

**Extensions**

* 2a. List of companies is empty.

    Use case ends.

**Use case: View company**

**MSS**

1. User requests to list companies.
2. User requests to view all jobs and contacts associated with a company.
3. TalentConnect filters the contact and job lists to show those associated with company.

    Use case ends.

**Extensions**
* 1a. The list is empty.

    Use case ends.

* 2a. The given parameters are invalid.
    * 2a1. TalentConnect shows an error message.
    
        Use case resumes at step 2.

### Non-Functional Requirements

1.  Should work on any _mainstream OS_ as long as it has Java `17` or above installed.
2.  Should be able to hold up to 1000 persons without a noticeable sluggishness in performance for typical usage.
3.  A user with above average typing speed for regular English text (i.e. not code, not system admin commands) should be able to accomplish most of the tasks faster using commands than using the mouse.

*{More to be added}*

### Glossary

* **Mainstream OS**: Windows, Linux, Unix, MacOS

--------------------------------------------------------------------------------------------------------------------

## **Appendix: Instructions for manual testing**

Given below are instructions to test the app manually.

<div markdown="span" class="alert alert-info">:information_source: **Note:** These instructions only provide a starting point for testers to work on;
testers are expected to do more *exploratory* testing.

</div>

### Launch and shutdown

1. Initial launch

   1. Download the jar file and copy into an empty folder

   1. Double-click the jar file Expected: Shows the GUI with a set of sample contacts. The window size may not be optimum.

1. Saving window preferences

   1. Resize the window to an optimum size. Move the window to a different location. Close the window.

   1. Re-launch the app by double-clicking the jar file.<br>
       Expected: The most recent window size and location is retained.

1. _{ more test cases …​ }_

### Deleting a contact

1. Deleting a contact while all contacts are being shown

   1. Prerequisites: List all persons using the `list` command. Multiple persons in the list.

   1. Test case: `delete contact 1`<br>
      Expected: First contact is deleted from the list. Details of the deleted contact shown in the status message.

   1. Test case: `delete contact 0`<br>
      Expected: No person is deleted. Error details shown in the status message. Status bar remains the same.

   1. Other incorrect delete commands to try: `delete`, `delete test x`, `...` (where x is larger than the list size)<br>
      Expected: Similar to previous.

1. _{ more test cases …​ }_

### Saving data

1. Dealing with missing/corrupted data files

   1. _{explain how to simulate a missing/corrupted file, and the expected behavior}_

1. _{ more test cases …​ }_

--------------------------------------------------------------------------------------------------------------------

## **Appendix: Planned Enhancements**

Team size: 5

* **Improve `screen job` functionality**: Currently, this command is only able to screen contacts that have roles which is a sub-string of the job name. 
We plan to expand on this functionality by allowing users to add parameters to modify the way screen works. Examples being to 
screen by how well the contact's skill match up to the job's requirements or to specify whether partial role matches are fine. 
* **Enable support for `find job`**:   
* **Enable support for `find company`**  
* **Enable support for `edit job`**  
* **Enable support for `edit company`**  
* **Enable support for job quota for `Job`**  
* **Enable support for more billing date formats (from end of month)**
