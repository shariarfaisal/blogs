# Reduce PDF File Size in Linux

```UNIX/Linux```

In our Linux system, If we have a large PDF file, we may want to reduce it’s size. We shall look at different ways to reduce PDF size or compress PDF files in Linux in this tutorial.


Let’s find out some Command Line and GUI methods to deal with this problem.



# Command Line Utilities to Reduce PDF File Size in Linux


## 1. Using GhostScript


We can use the ghostscript command line utility in Linux to compress PDFs.


If the command is not available in your machine, you can install it using your package manager.


For example, in Ubuntu, you can use apt:


```
sudo apt install ghostscript

```


You can use this magic command to compress PDFs to a readable quality.


```
gs -sDEVICE=pdfwrite -dCompatibilityLevel=1.4 -dPDFSETTINGS=/screen -dNOPAUSE -dQUIET -dBATCH -sOutputFile=output.pdf input.pdf

```


Here, replace output.pdf and input.pdf accordingly.


The various tweaks to the -dPDFSETTINGS option are provided in the table below. Use them according to your need.











-dPDFSETTINGS Option
Description


-dPDFSETTINGS=/screen
Has a lower quality and smaller size. (72 dpi)


-dPDFSETTINGS=/ebook
Has a better quality, but has a slightly larger size (150 dpi)


-dPDFSETTINGS=/prepress
Output is of a higher size and quality (300 dpi)


-dPDFSETTINGS=/printer
Output is of a printer type quality (300 dpi)


-dPDFSETTINGS=/default
Selects the output which is useful for multiple purposes. Can cause large PDFS.




I have used the above command to achieve a compression from 73MB to 14MB!


Ghostscript Reduce Pdf Size

## 2. Use ps2pdf


This command ps2pdf converts a PDF to PS and then again back, compressing it efficiently as a result.


It may not always work, but it can give very good results.


Format:


```
ps2pdf input.pdf output.pdf

```


It is recommended that you use the -dPDFSETTINGS=/ebooks setting to get the best performance, as ebooks have the best size for readability and also are small enough in size.


```
ps2pdf -dPDFSETTINGS=/ebook input.pdf output.pdf

```


I have tried this on a 73MB PDF and it had the same results as the ghostscript command, the compressed PDF having only 14MB!


Ps2pdf Reduce Pdf Size

# GUI Utilities to Reduce PDF File Size in Linux


If you are uncomfortable with using command line tools, there is a GUI alternative as well.


# Densify


This is a GUI front end to ghostscript, which can be installed in any Linux distribution, since it uses Python3 and it’s GTK modules.


This package is called Densify, and is available here(Link to github).


I have created a simple bash script to do all the necessary work. Run this bash script as root, to link and download necessary files.


```
#!/bin/bash
#- HELPER SCRIPT FOR DENSIFY
#-    original package         https://github.com/hkdb/Densify
#-    script author            Vijay Ramachandran
#-    site                     https://journaldev.com
#- 

# Go to your home directory (preferred)
cd $HOME

# Download the package
git clone https://github.com/hkdb/Densify
cd Densify

# Queue must be changed to queue in the file.
# Will not work otherwise
sed -i 's/Queue/queue/g' $PWD/densify

# Create the symlink to /opt
sudo ln -s $PWD /opt/Densify

# Perform the install
cd /opt/Densify
sudo chmod 755 install.sh
sudo ./install.sh

# Export to PATH
if [ $SHELL == "/bin/zsh" ]; then
    if test -f $HOME/.zshrc; then
        echo 'export PATH=/opt/Densify:$PATH' >> $HOME/.zshrc
        source $HOME/.zshrc
    else
        echo "No zshrc Found! Please create a zsh config file and try again"
    fi
else
    if [ $SHELL == "/bin/bash" ]; then
        if test -f $HOME/.bashrc; then
            echo 'export PATH=/opt/Densify:$PATH' >> $HOME/.bashrc
            source $HOME/.bashrc
        else
            if test -f $HOME/.bash_profile; then
                echo 'export PATH=/opt/Densify:$PATH' >> $HOME/.bash_profile
                source $HOME/.bash_profile
            else
                echo "No bashrc Found! Please create a bash config file and try again"
            fi
        fi
    else
        echo "Default Shell is not zsh or bash. Please add /opt/Densify to your PATH"
    fi
fi

```


If there are no errors, you are good to go! Simply type the below command from opt/densify to invoke the GUI, or open it from your dashboard.


```
densify

```


Densify Gui Utility
You can now compress as many PDF files as you need, using a GUI!



# References


- StackOverflow question on reducing PDF size


