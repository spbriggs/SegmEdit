# SegmEditGUI

## Introduction

SegmEditGUI is a desktop application useful for browsing and editing XML files
in TrueViz format. SegmEditGUI is written in Python language.

The main motivation for creating SegmEditGUI was to perform manual correction of
a set of TrueViz files holding the internal structure of scientific articles.
SegmEditGUI is intuitive and simple to use. It allows the user to perform
typical operations correcting the geometrical structure of a document.
 
## Requirements and running

SegmEditGUI runs on both Linux and MS Windows. It requires:

* Python 2.6 or 2.7
* python-wxgtk2.8
* ImageMagick
* GhostScript

The application does not require any compilation or installation phase. To run
the program, segmedit.py needs to be executed:

    ./segmedit.py

The application requires a local directory to store temporary files. If the
directory does not exist, it will be automatically created (more information in
the Configuration section).

## Working with files

SegmEditGUI uses two kinds of data files: PDF files and associated TrueViz
files, that store the geometrical structure extracted from the content of a PDF
file.

SegmEditGUI can be used as a standalone application, in which case all data
files are stored locally. Files can be also distributed over the network by a
separate SegmServer instance.

### Files stored locally

To begin work with a local file, the user should open two files:

* XML TrueViz file (`File -> Open local XML`),
* (optional) PDF file (`File -> Open PDF`).

If the wrong PDF file has been opened, another file can be opened without
re-opening the XML file.

There are two locations automatically searched by the application for a PDF file
associated with opened XML file: the directory of the XML file and the directory
of previously opened PDF file. If the application finds a PDF file with same
name as the XML file (except the extension) in one of those locations, the PDF
file will be opened automatically.

After performing all operations on XML file it should be saved (`File -> Save
XML/Save XML as...`).

Closing the XML file will result in closing the PDF file. The application will
show a warning message if the user tries to close an unsaved file.

### Network files

Files browsed and edited in SegmEditGUI can be distributed over the network by
SegmServer instance, instead of being stored locally. This allows to manage the
process of editing a set of documents by a group of users using their own
SegmEditGUI instances.

To manage distributing files, there are a few pieces of information stored by
the server with every file:

* document's id,
* document's title,
* document's status,
* optional comment about the document.

Possible statuses are:

* unlocked - the document is available to any user (default),
* locked - the document has been sent to one particular user and is being
processed,
* complete - processing complete, the user has sent the document back,
* error - error occurred.

To start working with a network document, the user should open it (`File -> Open
network document`). Before opening the first network file the application will
ask for a name to identify the user. It is a very simple identification
mechanism, that does not include any authentication process. Next, a window
presenting two lists of available documents will appear: the list of previously
opened documents and the list holding documents that have not been processed
yet. The window displays documents ids, titles and statuses.

After performing all operations on the file, it should be sent back to the
server. This can be done by choosing `File -> Send network file` in the
application menu. The user sending the document to a server should set its
status and optionally provide a comment message. Any status can be set: locked
(if the user plans to continue processing the document), unlocked (if the user
resigns from processing the document), complete (when the processing has been
completed) or error (if there are some problems with processing the document).

The server keeps track of the last user that processed given document (for
documents with statuses: locked, complete and error).

## Processing the document

### Geometrical structure

The application can be used for editing TrueViz files holding the geometrical
structure of the content of associated PDF file. Such a hierarchical structure
includes:

* individual characters,
* words containing characters,
* lines containing words,
* zones containing lines,
* pages containing zones.

All characters, words, lines and zones can be described by their content and
bounding box (a rectangle enclosing the object on the document's page).

## Navigation

While working on a file, the application displays the PDF file in the background
and the structure information from XML file on top of it (in the form of
semi-transparent rectangles).

The application allows to navigate through document's pages (using controls in
the left lower corner of the application window) and zoom the page (using
controls in the right lower corner of the application window).

## Undo and redo

The application keeps track of all performed operations, and allows the user to
undo every operation and redo the operation previously undone. This can be
achieved using the options from Edit menu or shortcuts Ctrl-Z and Ctrl-Y.

## Changing document's layers and work modes

SegmEditGUI allows to edit words, lines and zone layers. Current document layer
can be chosen using options at the bottom of the screen. When a particular layer
is chosen, the window displays the elements of this layer (in the form of
semi-transparent rectangles) and also the elements of upper and lower layer.
From this moment all operations on the elements of current layer are available,
but it is important to keep in mind that some operations can affect the elements
from other layers as well.

There are also three work modes available to the user:

* merge, useful for merging elements,
* cut out, that allows to create new elements by cutting out parts of other
elements,
* classification, useful for classifying zones.

Work mode can be chosen using options at the bottom of the screen, Mode menu or
Ctrl-M shortcut.

### Merge mode

Merge mode allows to create a new element by merging a few smaller ones. To do
this, the user needs to point all the objects to merge using mouse (drag and
drop). Of course merging words, that belong to different lines, will also cause
merging those lines as well, and the same issue applies to merging lines.
Created element will have minimal dimensions enclosing all the lower-layer
elements that belong to it.

### Cut out mode

Cut out operation can be used to create a new element out of fragments of other
elements. The detailed description of the cut out operation on words is given
below. For other layers the mechanism is analogous.

The user uses mouse (drag and drop) to "draw" a rectangle enclosing a group of
characters. The application creates a new word, that contains all characters
that overlap with the rectangle pointed by the user. The dimensions of the new
word and words, that are about to shrink, are properly recalculated. If there is
a word containing no characters, it is removed.

### Classification mode

Classification mode allows to classify zones and thus can be used only in the
zone layer. Choosing this mode results in automatic switching to zone layer.
Choosing other layer in classification mode will cause switching to merge mode.
In classification mode every zone is displayed using the colour denoting its
label.

To classify a zone, the user needs to right-click on the chosen zone and choose
from available label names.

It is also possible to set the label of many zones at the same time. To do this,
the user needs to choose a group of zones using mouse (drag and drop), and again
choose a label from label list.

### Additional operations

There are two additional operations available by using right-click on the
element: Atomize (all layers) and Turn word into line (only the word layer).
Both can be equally used in merge and cut out modes.

Atomize operation can be used to divide the element into the smallest fragments
possible, without interfering in the lower layer. For example performing Atomize
on a word will divide the word into 1-character words.

"Turn word into line" was implemented as fast and very useful operation for
correcting common segmentation errors. It consists of two operations:

* creating a new line containing only one chosen word,
* dividing the word into a several smaller words based on spaces.

Before performing this operation, all words of the same line should be joined
into one large word.

## Configuration

SegmEditGUI contains a single configuration file `config.py`, that can be used
to set numerous options determining: the look of the application, paths and
addresses, the label set. While changing options values the syntax of Python
language should be preserved.

The most important options include:

* VAR_DIRECTORY - path to the directory used by the application to store all
temporary files. Warning: the application does not remove temporary files, so in
case of limited disk space the directory should be purged occasionally.
* NETWORK_ADDRESS - the address and port number of the SegmServer instance, which
will distribute network files to the application.
* ZONE_LABELS -- the list of labels, that can be associated to zones. Example list
element:

    ZoneLabel('author', 'Author', ShapeStyle((255, 180, 180, 90)))

  where 'author' is the label name stored in the TrueViz file, 'Author' is the
  description shown in the application and the numbers stand for the color (RGB)
  and transparency of the rectangle representing the zone in the classification
  mode.
