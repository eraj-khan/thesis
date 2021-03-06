# -*- mode: python3 -*-
# SCons build file

import re
import os
import subprocess

BASEFILE = "thesis"

TEXFILE = BASEFILE + ".tex"
PDFFILE = BASEFILE + ".pdf"


# Regex for getting the precise window created by mupdf and listed by wmctrl
WMCTRL_REGEX = re.compile(".+({}.+72 dpi\))".format(PDFFILE))


# A utility function for finding substrings inside strings. Return true if the substring is present:
def contains(word, line):
    return line.find(word) >= 0


# A Generator for showing the pdf (passed in as the 'source')
# The String returned by the generator is executed by scons as a shell command
def show_pdf(target, source, env, for_signature):
    process = subprocess.Popen(['which', 'evince'], stdout=subprocess.PIPE, stderr=subprocess.PIPE)
    out, err = process.communicate()

    if "evince" in out:
        return "evince {} &".format(source[0])
    else:
        # Determine if the pdf file is already open in mupdf
        out = subprocess.check_output(['wmctrl', '-l'])

        if contains(PDFFILE, out):                           # True if the PDFFILE is already open in which case it shows up in wmctrl and out.find() returns a semi-positive number for the position of the substring

            # The if above only tests for the word "thesis.pdf" which could be the name of a window other than mupdf. To that end we look for a line matching WMCTR_REGEX
            for line in out.split('\n'):

                m = WMCTRL_REGEX.match(line)

                # The matched substring provides a more specific way of targeting mupdf only
                if m:
                    return 'wmctrl -R "{}" && xdotool key r'.format(m.group(1))

        # If nothing is found in wmctrl we launch the file from scratch
        return 'mupdf {} &'.format(PDFFILE)

# We create a custom builder that uses the show_pdf generator to show the specified file
show_builder = Builder(generator = show_pdf)

# We use ENV= to append the os environmental variables to scons since we need access to the path to get pdflatex
# We also append the show_builder under the name 'Show'.
env = Environment(ENV=os.environ, BUILDERS={'Show': show_builder})

# Pass pdflatex flags to scons
env.AppendUnique(PDFLATEXFLAGS='-synctex=-1')       # Generates the .synctex index that allows clicking in the pdf to open up corresponding code section in Atom
env.AppendUnique(PDFLATEXFLAGS='-shell-escape')     # Allows running pdflatex in multiple shells. Speeds up building of tikz images.
env.AppendUnique(PDFLATEXFLAGS='-halt-on-error')    # Halts compilation on the first error discovered. Speeds up development and debugging


# The env.PDF declares that report.pdf is to created from (depends upon) report.tex and is build using pdflatex
pdf = env.PDF(target=PDFFILE, source=TEXFILE)

# Declare additional files to be cleaned when 'scons -c' is run
env.Clean(pdf, ['thesis.run.xml', 'thesis.synctex'])


# We create an instance of the Show builder and make it the primary dependency of this project. The rest are connected to it through the builders defined above
show = env.Show(PDFFILE)
Depends(show, pdf)
Default(show)
