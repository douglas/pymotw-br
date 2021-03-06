=============================================
fileinput -- Process lines from input streams
=============================================

.. module:: fileinput
    :synopsis: Process lines from input streams.

:Purpose: Create command-line filter programs to process lines from input streams.
:Available In: 1.5.2 and later

The fileinput module is a framework for creating command line programs
for processing text files in a filter-ish manner. 

Converting M3U files to RSS
===========================

For example, the
m3utorss_ app I recently wrote for my friend `Patrick
<http://events.mediumloud.com/>`_ to convert some of his demo
recordings into a podcastable format.

The inputs to the program are one or more m3u file listing the mp3 files to be
distributed. The output is a single blob of XML that looks like an RSS feed
(output is written to stdout, for simplicity). To process the input, I need to
iterate over the list of filenames and:

* Open each file.
* Read each line of the file.
* Figure out if the line refers to an mp3 file.
* If it does, extract the information from the mp3 file needed for the RSS feed.
* Print the output.

I could have written all of that file handling out by hand. It isn't that
complicated, and with some testing I'm sure I could even get the error
handling right. But with the fileinput module, I don't need to worry about
that. I just write something like:

.. literalinclude:: fileinput_example.py
   :lines: 30-37

The ``fileinput.input()`` function takes as argument a list of
filenames to examine. If the list is empty, the module reads data from
standard input. The function returns an iterator which returns
individual lines from the text files being processed.  So, all I have
to do is loop over each line, skipping blanks and comments, to find
the references to mp3 files.

Here's the complete program:

.. include:: fileinput_example.py
   :literal:
   :start-after: #end_pymotw_header

and its output:

.. {{{cog
.. cog.out(run_script(cog.inFile, 'fileinput_example.py sample_data.m3u'))
.. }}}
.. {{{end}}}

Progress Meta-data
==================

In the previous example, I did not care what file or line number we
are processing in the input. For other tools (grep-like searching, for
example) you might. The fileinput module includes functions for
accessing that information (``filename()``, ``filelineno()``,
``lineno()``, etc.).

.. include:: fileinput_grep.py
   :literal:
   :start-after: #end_pymotw_header

We can use this basic pattern matching loop to find the occurances of
"fileinput" in the source for the examples.

.. {{{cog
.. cog.out(run_script(cog.inFile, 'fileinput_grep.py fileinput *.py'))
.. }}}
.. {{{end}}}

We can also pass input to it through stdin.

.. {{{cog
.. cog.out(run_script(cog.inFile, 'cat *.py | python fileinput_grep.py fileinput', interpreter=None))
.. }}}
.. {{{end}}}


In-place Filtering
==================

Another common file processing operation is to modify the contents.
For example, a Unix hosts file might need to be updated if a subnet
range changes.

.. include:: etc_hosts
   :literal:

The safe way to make the change automatically is to create a new file
based on the input and then replace the original with the edited copy.
fileinput supports this automatically using the *inplace* option.

.. include:: fileinput_change_subnet.py
   :literal:
   :start-after: #end_pymotw_header

.. {{{cog
.. path('PyMOTW/fileinput/etc_hosts').copy('PyMOTW/fileinput/etc_hosts.txt')
.. cog.out(run_script(cog.inFile, 'fileinput_change_subnet.py 172.16.177 172.16.178 etc_hosts.txt'))
.. }}}
.. {{{end}}}

Although the script uses ``print``, no output is produced to stdout
because fileinput maps stdout to the file being overwritten.

.. include:: etc_hosts.txt
   :literal:

Before processing begins, a backup file is created using the original
name plus ``.bak``.  The backup file is removed when the input is
closed.

.. include:: fileinput_change_subnet_noisy.py
   :literal:
   :start-after: #end_pymotw_header

.. {{{cog
.. path('PyMOTW/fileinput/etc_hosts').copy('PyMOTW/fileinput/etc_hosts.txt')
.. cog.out(run_script(cog.inFile, 'fileinput_change_subnet_noisy.py 172.16.177 172.16.178 etc_hosts.txt'))
.. }}}
.. {{{end}}}


.. seealso::

    `fileinput <http://docs.python.org/library/fileinput.html>`_
        The standard library documentation for this module.

    `Patrick Bryant <http://events.mediumloud.com/>`_
        Atlanta-based singer/song-writer.

    m3utorss_
        Script to convert m3u files listing MP3s to an RSS file
        suitable for use as a podcast feed.

    :ref:`xml.etree.ElementTree.creating`
        More details of using ElementTree to produce XML.

    :ref:`article-file-access`
        Other modules for working with files.

    :ref:`article-text-processing`
        Other modules for working with text.

.. _m3utorss: http://www.doughellmann.com/projects/m3utorss
