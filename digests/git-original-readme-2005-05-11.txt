


	GIT - the stupid content tracker


"git" can mean anything, depending on your mood.

 - random three-letter combination that is pronounceable, and not
   actually used by any common UNIX command.  The fact that it is a
   mispronunciation of "get" may or may not be relevant.
 - stupid. contemptible and despicable. simple. Take your pick from the
   dictionary of slang.
 - "global information tracker": you're in a good mood, and it actually
   works for you. Angels sing, and a light suddenly fills the room. 
 - "goddamn idiotic truckload of sh*t": when it breaks

This is a stupid (but extremely fast) directory content manager.  It
doesn't do a whole lot, but what it _does_ do is track directory
contents efficiently. 

There are two object abstractions: the "object database", and the
"current directory cache" aka "index".



	The Object Database (GIT_OBJECT_DIRECTORY)


The object database is literally just a content-addressable collection
of objects.  All objects are named by their content, which is
approximated by the SHA1 hash of the object itself.  Objects may refer
to other objects (by referencing their SHA1 hash), and so you can build
up a hierarchy of objects. 

All objects have a statically determined "type" aka "tag", which is
determined at object creation time, and which identifies the format of
the object (i.e. how it is used, and how it can refer to other objects). 
There are currently three different object types: "blob", "tree" and
"commit". 

A "blob" object cannot refer to any other object, and is, like the tag
implies, a pure storage object containing some user data.  It is used to
actually store the file data, i.e. a blob object is associated with some
particular version of some file. 

A "tree" object is an object that ties one or more "blob" objects into a
directory structure. In addition, a tree object can refer to other tree
objects, thus creating a directory hierarchy. 

Finally, a "commit" object ties such directory hierarchies together into
a DAG of revisions - each "commit" is associated with exactly one tree
(the directory hierarchy at the time of the commit). In addition, a
"commit" refers to one or more "parent" commit objects that describe the
history of how we arrived at that directory hierarchy.

As a special case, a commit object with no parents is called the "root"
object, and is the point of an initial project commit.  Each project
must have at least one root, and while you can tie several different
root objects together into one project by creating a commit object which
has two or more separate roots as its ultimate parents, that's probably
just going to confuse people.  So aim for the notion of "one root object
per project", even if git itself does not enforce that. 

Regardless of object type, all objects are share the following
characteristics: they are all in deflated with zlib, and have a header
that not only specifies their tag, but also size information about the
data in the object.  It's worth noting that the SHA1 hash that is used
to name the object is always the hash of this _compressed_ object, not
the original data.

As a result, the general consistency of an object can always be tested
independently of the contents or the type of the object: all objects can
be validated by verifying that (a) their hashes match the content of the
file and (b) the object successfully inflates to a stream of bytes that
forms a sequence of <ascii tag without space> + <space> + <ascii decimal
size> + <byte\0> + <binary object data>. 

The structured objects can further have their structure and connectivity
to other objects verified. This is generally done with the "fsck-cache"
program, which generates a full dependency graph of all objects, and
verifies their internal consistency (in addition to just verifying their
superficial consistency through the hash).

The object types in some more detail:

  BLOB: A "blob" object is nothing but a binary blob of data, and
	doesn't refer to anything else.  There is no signature or any
	other verification of the data, so while the object is
	consistent (it _is_ indexed by its sha1 hash, so the data itself
	is certainly correct), it has absolutely no other attributes. 
	No name associations, no permissions.  It is purely a blob of
	data (i.e. normally "file contents"). 

	In particular, since the blob is entirely defined by its data,
	if two files in a directory tree (or in multiple different
	versions of the repository) have the same contents, they will
	share the same blob object. The object is totally independent
	of it's location in the directory tree, and renaming a file does
	not change the object that file is associated with in any way.

  TREE: The next hierarchical object type is the "tree" object.  A tree
	object is a list of mode/name/blob data, sorted by name. 
	Alternatively, the mode data may specify a directory mode, in
	which case instead of naming a blob, that name is associated
	with another TREE object. 

	Like the "blob" object, a tree object is uniquely determined by
	the set contents, and so two separate but identical trees will
	always share the exact same object. This is true at all levels,
	i.e. it's true for a "leaf" tree (which does not refer to any
	other trees, only blobs) as well as for a whole subdirectory.

	For that reason a "tree" object is just a pure data abstraction:
	it has no history, no signatures, no verification of validity,
	except that since the contents are again protected by the hash
	itself, we can trust that the tree is immutable and its contents
	never change. 

	So you can trust the contents of a tree to be valid, the same
	way you can trust the contents of a blob, but you don't know
	where those contents _came_ from.

	Side note on trees: since a "tree" object is a sorted list of
	"filename+content", you can create a diff between two trees
	without actually having to unpack two trees.  Just ignore all
	common parts, and your diff will look right.  In other words,
	you can effectively (and efficiently) tell the difference
	between any two random trees by O(n) where "n" is the size of
	the difference, rather than the size of the tree. 

	Side note 2 on trees: since the name of a "blob" depends
	entirely and exclusively on its contents (i.e. there are no names
	or permissions involved), you can see trivial renames or
	permission changes by noticing that the blob stayed the same. 
	However, renames with data changes need a smarter "diff" implementation. 

CHANGESET: The "changeset" object is an object that introduces the
	notion of history into the picture.  In contrast to the other
	objects, it doesn't just describe the physical state of a tree,
	it describes how we got there, and why. 

	A "changeset" is defined by the tree-object that it results in,
	the parent changesets (zero, one or more) that led up to that
	point, and a comment on what happened.  Again, a changeset is
	not trusted per se: the contents are well-defined and "safe" due
	to the cryptographically strong signatures at all levels, but
	there is no reason to believe that the tree is "good" or that
	the merge information makes sense.  The parents do not have to
	actually have any relationship with the result, for example. 

	Note on changesets: unlike real SCM's, changesets do not contain
	rename information or file mode change information.  All of that
	is implicit in the trees involved (the result tree, and the
	result trees of the parents), and describing that makes no sense
	in this idiotic file manager. 

TRUST: The notion of "trust" is really outside the scope of "git", but
	it's worth noting a few things.  First off, since everything is
	hashed with SHA1, you _can_ trust that an object is intact and
	has not been messed with by external sources.  So the name of an
	object uniquely identifies a known state - just not a state that
	you may want to trust. 

	Furthermore, since the SHA1 signature of a changeset refers to
	the SHA1 signatures of the tree it is associated with and the
	signatures of the parent, a single named changeset specifies
	uniquely a whole set of history, with full contents.  You can't
	later fake any step of the way once you have the name of a
	changeset. 

	So to introduce some real trust in the system, the only thing
	you need to do is to digitally sign just _one_ special note,
	which includes the name of a top-level changeset.  Your digital
	signature shows others that you trust that changeset, and the
	immutability of the history of changesets tells others that they
	can trust the whole history. 

	In other words, you can easily validate a whole archive by just
	sending out a single email that tells the people the name (SHA1
	hash) of the top changeset, and digitally sign that email using
	something like GPG/PGP. 

	In particular, you can also have a separate archive of "trust
	points" or tags, which document your (and other peoples) trust. 
	You may, of course, archive these "certificates of trust" using
	"git" itself, but it's not something "git" does for you. 

Another way of saying the last point: "git" itself only handles content
integrity, the trust has to come from outside. 



	The "index" aka "Current Directory Cache" (".git/index")


The index is a simple binary file, which contains an efficient
representation of a virtual directory content at some random time.  It
does so by a simple array that associates a set of names, dates,
permissions and content (aka "blob") objects together.  The cache is
always kept ordered by name, and names are unique (with a few very
specific rules) at any point in time, but the cache has no long-term
meaning, and can be partially updated at any time. 

In particular, the index certainly does not need to be consistent with
the current directory contents (in fact, most operations will depend on
different ways to make the index _not_ be consistent with the directory
hierarchy), but it has three very important attributes:

 (a) it can re-generate the full state it caches (not just the directory
     structure: it contains pointers to the "blob" objects so that it
     can regenerate the data too)

     As a special case, there is a clear and unambiguous one-way mapping
     from a current directory cache to a "tree object", which can be
     efficiently created from just the current directory cache without
     actually looking at any other data.  So a directory cache at any
     one time uniquely specifies one and only one "tree" object (but
     has additional data to make it easy to match up that tree object
     with what has happened in the directory)

 (b) it has efficient methods for finding inconsistencies between that
     cached state ("tree object waiting to be instantiated") and the
     current state. 

 (c) it can additionally efficiently represent information about merge
     conflicts between different tree objects, allowing each pathname to
     be associated with sufficient information about the trees involved
     that you can create a three-way merge between them.

Those are the three ONLY things that the directory cache does.  It's a
cache, and the normal operation is to re-generate it completely from a
known tree object, or update/compare it with a live tree that is being
developed.  If you blow the directory cache away entirely, you generally
haven't lost any information as long as you have the name of the tree
that it described. 

At the same time, the directory index is at the same time also the
staging area for creating new trees, and creating a new tree always
involves a controlled modification of the index file.  In particular,
the index file can have the representation of an intermediate tree that
has not yet been instantiated.  So the index can be thought of as a
write-back cache, which can contain dirty information that has not yet
been written back to the backing store. 



	The Workflow


Generally, all "git" operations work on the index file. Some operations
work _purely_ on the index file (showing the current state of the
index), but most operations move data to and from the index file. Either
from the database or from the working directory. Thus there are four
main combinations: 

 1) working directory -> index

	You update the index with information from the working directory
	with the "update-cache" command.  You generally update the index
	information by just specifying the filename you want to update,
	like so:

		update-cache filename

	but to avoid common mistakes with filename globbing etc, the
	command will not normally add totally new entries or remove old
	entries, i.e. it will normally just update existing cache entries.

	To tell git that yes, you really do realize that certain files
	no longer exist in the archive, or that new files should be
	added, you should use the "--remove" and "--add" flags
	respectively.

	NOTE! A "--remove" flag does _not_ mean that subsequent
	filenames will necessarily be removed: if the files still exist
	in your directory structure, the index will be updated with
	their new status, not removed. The only thing "--remove" means
	is that update-cache will be considering a removed file to be a
	valid thing, and if the file really does not exist any more, it
	will update the index accordingly. 

	As a special case, you can also do "update-cache --refresh",
	which will refresh the "stat" information of each index to match
	the current stat information. It will _not_ update the object
	status itself, and it will only update the fields that are used
	to quickly test whether an object still matches its old backing
	store object.

 2) index -> object database

	You write your current index file to a "tree" object with the
	program

		write-tree

	that doesn't come with any options - it will just write out the
	current index into the set of tree objects that describe that
	state, and it will return the name of the resulting top-level
	tree. You can use that tree to re-generate the index at any time
	by going in the other direction:

 3) object database -> index

	You read a "tree" file from the object database, and use that to
	populate (and overwrite - don't do this if your index contains
	any unsaved state that you might want to restore later!) your
	current index.  Normal operation is just

		read-tree <sha1 of tree>

	and your index file will now be equivalent to the tree that you
	saved earlier. However, that is only your _index_ file: your
	working directory contents have not been modified.

 4) index -> working directory

	You update your working directory from the index by "checking
	out" files. This is not a very common operation, since normally
	you'd just keep your files updated, and rather than write to
	your working directory, you'd tell the index files about the
	changes in your working directory (i.e. "update-cache").

	However, if you decide to jump to a new version, or check out
	somebody else's version, or just restore a previous tree, you'd
	populate your index file with read-tree, and then you need to
	check out the result with

		checkout-cache filename

	or, if you want to check out all of the index, use "-a".

	NOTE! checkout-cache normally refuses to overwrite old files, so
	if you have an old version of the tree already checked out, you
	will need to use the "-f" flag (_before_ the "-a" flag or the
	filename) to _force_ the checkout.


Finally, there are a few odds and ends which are not purely moving from
one representation to the other:

 5) Tying it all together

	To commit a tree you have instantiated with "write-tree", you'd
	create a "commit" object that refers to that tree and the
	history behind it - most notably the "parent" commits that
	preceded it in history. 

	Normally a "commit" has one parent: the previous state of the
	tree before a certain change was made. However, sometimes it can
	have two or more parent commits, in which case we call it a
	"merge", due to the fact that such a commit brings together
	("merges") two or more previous states represented by other
	commits. 

	In other words, while a "tree" represents a particular directory
	state of a working directory, a "commit" represents that state
	in "time", and explains how we got there. 

	You create a commit object by giving it the tree that describes
	the state at the time of the commit, and a list of parents:

		commit-tree <tree> -p <parent> [-p <parent2> ..]

	and then giving the reason for the commit on stdin (either
	through redirection from a pipe or file, or by just typing it at
	the tty). 

	commit-tree will return the name of the object that represents
	that commit, and you should save it away for later use.
	Normally, you'd commit a new "HEAD" state, and while git doesn't
	care where you save the note about that state, in practice we
	tend to just write the result to the file ".git/HEAD", so that
	we can always see what the last committed state was.

 6) Examining the data

	You can examine the data represented in the object database and
	the index with various helper tools. For every object, you can
	use "cat-file" to examine details about the object:

		cat-file -t <objectname>

	shows the type of the object, and once you have the type (which
	is usually implicit in where you find the object), you can use

		cat-file blob|tree|commit <objectname>

	to show its contents. NOTE! Trees have binary content, and as a
	result there is a special helper for showing that content,
	called "ls-tree", which turns the binary content into a more
	easily readable form.

	It's especially instructive to look at "commit" objects, since
	those tend to be small and fairly self-explanatory. In
	particular, if you follow the convention of having the top
	commit name in ".git/HEAD", you can do

		cat-file commit $(cat .git/HEAD)

	to see what the top commit was.

 7) Merging multiple trees

	Git helps you do a three-way merge, which you can expand to
	n-way by repeating the merge procedure arbitrary times until you
	finally "commit" the state.  The normal situation is that you'd
	only do one three-way merge (two parents), and commit it, but if
	you like to, you can do multiple parents in one go.

	To do a three-way merge, you need the two sets of "commit"
	objects that you want to merge, use those to find the closest
	common parent (a third "commit" object), and then use those
	commit objects to find the state of the directory ("tree"
	object) at these points. 

	To get the "base" for the merge, you first look up the common
	parent of two commits with

		merge-base <commit1> <commit2>

	which will return you the commit they are both based on.  You
	should now look up the "tree" objects of those commits, which
	you can easily do with (for example)

		cat-file commit <commitname> | head -1

	since the tree object information is always the first line in a
	commit object. 

	Once you know the three trees you are going to merge (the one
	"original" tree, aka the common case, and the two "result" trees,
	aka the branches you want to merge), you do a "merge" read into
	the index. This will throw away your old index contents, so you
	should make sure that you've committed those - in fact you would
	normally always do a merge against your last commit (which
	should thus match what you have in your current index anyway).
	To do the merge, do

		read-tree -m <origtree> <target1tree> <target2tree>

	which will do all trivial merge operations for you directly in
	the index file, and you can just write the result out with
	"write-tree". 

	NOTE! Because the merge is done in the index file, and not in
	your working directory, your working directory will no longer
	match your index. You can use "checkout-cache -f -a" to make the
	effect of the merge be seen in your working directory.

	NOTE2! Sadly, many merges aren't trivial. If there are files
	that have been added.moved or removed, or if both branches have
	modified the same file, you will be left with an index tree that
	contains "merge entries" in it. Such an index tree can _NOT_ be
	written out to a tree object, and you will have to resolve any
	such merge clashes using other tools before you can write out
	the result.

	[ fixme: talk about resolving merges here ]

