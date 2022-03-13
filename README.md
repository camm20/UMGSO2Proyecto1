\documentclass{article}
\usepackage[utf8]{inputenc}
\usepackage{graphicx}
\usepackage{float}
\usepackage{listings}

\title{Proyecto 1 - SOPES2 \LaTeX{}}
\author{CÉSAR ARMANDO MORALES MARTÍNEZ}
\date{March 2022}


\begin{document}
\maketitle

\section{Introducción}

Se nos plantea un proyecto en el cual se cumplan las siguientes indicaciones:

El proyecto esta basado en la administración de servicios en un S.O. Unix, es necesario instalar
OpenBSD o FreeBSD (como host o virtual), instalar el servicio de MariaDB y un sitio Web
(recomendable usar un CMS para diseñar la página), configurar al menos dos usuarios para que
realizen lo siguiente:
a) Primer usuario: realiza una copia de respaldo de la base de datos y el sitio web todos los días a
las 03:00 horas (para la calificación se cambiará a cada minuto) y lo compiará a la
carpeta /tmp/backup/ con la estructura para el nombre umg-bkdb-ddMMyyyyHHmm.tar.gz y
umg-bkws-ddMMyyyyHHmm.tar.gz, donde la parte ddMMyyyyHHmm será la variable,

supóngase que la tarea se ejecutó en fecha 12/02/2022 03:00:21 los archivos se llamarán umg-
bkdb-12022022030021.tar.gz contendrá el archivo .sql que se generará al realizar la copia de

respando de la base de datos y umg-bkws-12022022030021.tar.gz que contendrá la copia de
respaldo del sitio web.
b) Segundo usuario: será el encargado de mover el archivo de respaldo al directorio controlado que
será /home/backup/ en este directorio, dicho usuario solo tendrá permisos de lectura y escritura,
por lo que si se ejecuta el comando rm -rf /home/backup/umg-bk-12022022030021.tar.gz
deberá mostrar error de permisos.

\section{Contenido}

En este caso se ha decidido utilizar una Raspberry Pi 3B+ con Raspbian como SO, en el cual vamos a realizar lo requerido para ello desarrollamos los siguientes pasos:

En el sistema se ha montado lo siguiente:
\begin{enumerate}
    \item APACHE
    \item MARIADB
    \item PHP
\end{enumerate}

Con las herramientas anteriores se ha montado un CMS (Wordpress) para la elaboración de un Sitio Web el cual cumplira la función de ser el sitio o aplicacion a la cual se le hara backup.

\textbf{**NOTA:} Se asume que ya se cuenta con una base de datos y un sitio web previamente configurados para el uso de esta guía.

\subsubsection{Creación de Usuarios:}
\begin{itemize}
     \item \verb|sudo adduser pi|
     
Este usuario sera el encargado de realizaro los backups en el directorio \verb|/tmp/backup|

\item \verb|sudo adduser cesar| 

Este usuario sera el encargado de mover los backups de \verb|/tmp/backup| a \verb|/home/backup|.

\end{itemize}

\subsubsection{Creación de Directorio \textbf{backup} en ḥome:}
\begin{itemize}
     \item \verb|sudo mkdir /home/backup|
     \item \verb|sudo chgrp cesar /home/backup|
     
*Se puede crear otro grupo desde linux pero en esta ocasión usaremos el grupo del usuario que se crea por defecto al crear el usuario

\item \verb|sudo chmod 775 /home/backup| 

* Esto se hace para que el grupo tenga permisos de lectura, escritura y ejecución

\end{itemize}

\subsubsection{Creación de File sh para el usuario \textbf{pi} quien creara los backups:}

Para esto creamos un archivo Shel el cual estara alojado en el path del usuario \textbf{pi} llamado \textbf{bkcreator.sh}:

\begin{lstlisting}[language=bash]
#!/bin/bash
#bkcreator.sh

now=$(date +%d%m%Y%H%M%S)

if [ ! -e /tmp/backup ]; then
	mkdir /tmp/backup
	chgrp cesar /tmp/backup
	chmod 775 /tmp/backup
fi

mysqldump -u sopes2 -psopes2 sopes2 > /tmp/backup/umg-bkdb-$now.sql
tar -czvf /tmp/backup/umg-bkdb-$now.tar.gz /tmp/backup/umg-bkdb-$now.sql
rm /tmp/backup/umg-bkdb-$now.sql
tar -czvf /tmp/backup/umg-bkws-$now.tar.gz /var/www/sopes2
\end{lstlisting}

\subsubsection{Creación de CRON para automatizar la tarea de creacion de los backups:}
\begin{itemize}
\item \verb|crontab -e|
\item \verb|0 3 * * * sh ~/bkcreator.sh|

Esto ejecutara el cron a las 03:00 de la creacion del backup.
\end{itemize}

\subsubsection{Creación de File sh para el usuario \textbf{cesar} quien movera los backups:}

Para esto creamos un archivo Shel el cual estara alojado en el path del usuario \textbf{cesar} llamado \textbf{bkcreator.sh}:

\begin{lstlisting}[language=bash]
#!/bin/bash
#bkmove.sh

if [ -e /tmp/backup ]; then
	nFiles=$(ls /tmp/backup | wc -l)
	if [ ! $nFiles -lt 1 ]; then
		mv /tmp/backup/* /home/backup
	fi
fi

\end{lstlisting}

\subsubsection{Creación de CRON para automatizar la tarea de movimiento de los backups:}
\begin{itemize}
\item \verb|crontab -e|
\item \verb|20 3 * * * sh ~/bkmove.sh|

Esto ejecutara el cron a las 03:20 para dar tiempo a la creacion del backup.
\end{itemize}

\section{GITHUB}

LINK \textbf{\textit{https://github.com/camm20/UMGSO2Proyecto1}}

\section{YOUTUBE}
LINK \textbf{\textit{https://youtu.be/-GIeXGGFK7o}}

\end{document}
