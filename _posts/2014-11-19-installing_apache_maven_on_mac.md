---
layout: post
title: Installing_Apache_Maven_on_Mac
categories: 开发环境
tags: apache mac
---

Download the Apache Maven bin.tar.gz file from http://maven.apache.org/download.cgi.
Extract the distribution archive to the directory where you want to install Maven, e.g. /Applications/apache-maven-3.1.1.

$ tar -xvf apache-maven-3.1.1-bin.tar.gz
Add the M2_HOME environment variable. Edit your ~/.bash_profile with any text editor. Add the following lines to the end of the file:

export M2_HOME=/Applications/apache-maven-3.1.1
export PATH=$PATH:$M2_HOME/bin
Restart the terminal and check the version installed with the following command:

$ mvn -version
The system displays information on the Apache Maven version installed.

