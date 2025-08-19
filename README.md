# An-Emulator-for-Smart-Radio-Environment
This project provides a Software Emulator for RIS-assisted wireless communication systems.  The emulator is designed to replicate the behavior of Software Defined Radios (SDRs) and RIS at the baseband and passband levels, allowing researchers and developers to evaluate wireless networks without requiring physical hardware. 

\documentclass[12pt]{article}
\usepackage{geometry}
\usepackage{hyperref}
\usepackage{enumitem}
\usepackage{amsmath}
\usepackage{amssymb}
\usepackage{listings}
\usepackage{dirtree}

\geometry{margin=1in}

\title{Emulator for RIS-Assisted Wireless Communication}
\date{}

\begin{document}

\maketitle

\section*{Overview}
This project provides a \textbf{Software Emulator} for RIS-assisted wireless communication systems. 
The emulator is designed to replicate the behavior of Software Defined Radios (SDRs) and Reconfigurable 
Intelligent Surfaces (RIS) at the baseband and passband levels, allowing researchers and developers 
to evaluate RIS-enabled wireless networks without requiring physical hardware.  

The emulator exposes standardized interfaces that closely resemble existing SDR APIs (e.g., USRP), 
making it straightforward to integrate with external baseband signal generators and RIS controllers. 
It supports a wide range of wireless channel models, realistic RF impairments, mobility models, and 
programmable RIS abstractions.  

Users can define nodes, RIS parameters, and the environment through structured \texttt{JSON} and 
\texttt{CSV} configuration files. The emulator then operates in discrete time steps to simulate signal 
transmission, reception, propagation, and RIS effects. All results, including received IQ samples and 
event logs, are stored in machine-readable formats for easy analysis and visualization.  

By providing a modular and extensible design, the emulator enables rapid prototyping, performance 
evaluation, and validation of RIS-assisted communication systems in a controlled yet realistic 
software environment.

\section*{Features}
\begin{itemize}[leftmargin=*]
    \item Emulates \textbf{RIS-assisted wireless communication channels}.
    \item Provides SDR-like APIs for seamless integration with external traffic generators.
    \item Supports \textbf{discrete-time emulation} with real-time control of RIS.
    \item Modular architecture for mobility models, channel models, and SDR impairments.
    \item JSON/CSV-based configuration for easy scenario setup.
    \item Outputs received IQ samples and event logs in structured JSON format.
\end{itemize}

\section*{System Design}
The Emulator consists of two major blocks:  
\begin{enumerate}
    \item \textbf{External Entities} (not part of emulator):
    \begin{itemize}
        \item Baseband signal generators
        \item RIS Controller
        \item Hardware RIS Driver
    \end{itemize}
    \item \textbf{Emulation Entities} (implemented in emulator):
    \begin{itemize}
        \item SDR Emulation
        \item Core Engine (mobility updates, request handling, channel modeling)
        \item RIS modeling and control
    \end{itemize}
\end{enumerate}

Communication between external entities and emulator is achieved via \textbf{Linux FIFO queues, named pipes, and JSON files}.  

\section*{Emulator Workflow}

\subsection*{Initialization Stage}
\begin{itemize}
    \item Loads system parameters from JSON files (\texttt{Nodes.json}, \texttt{Environment.json}, \texttt{RIS.json}).
    \item Loads RIS abstraction from CSV:  
    \[
      \{frequency, configuration\_id, phase\_response, gain\_response, angle\_incidence, angle\_reflection\}
    \]
    \item Users can configure:
    \begin{itemize}
        \item Node locations and mobility models (static, random waypoint, random direction, Gauss-Markov, random walk).
        \item RIS parameters (dimensions, orientation, reflection properties).
        \item RF impairments for each node.
    \end{itemize}
    \item Initializes IPC interfaces with unique identifiers per node/RIS.
\end{itemize}

\subsection*{Emulation Stage}
The emulator runs in discrete time steps of duration $\tau$:  

At each step, \texttt{Core/Engine.py} performs:
\begin{enumerate}
    \item \textbf{Request Handling} – Reads transmission/reception requests from external entities.
    \item \textbf{Mobility Update} – Updates node positions using mobility models.
    \item \textbf{Channel Emulation} – Generates received IQ samples, applies LOS/NLOS propagation, RIS effects, and RF impairments.
    \item \textbf{RIS Abstraction \& Control} – Updates RIS per-element coefficients based on user commands.\\
    \textbf{Note:} A dummyRSC file is included to demonstrate RIS control functionality. This file periodically sends configuration commands to the emulator at every discrete time step, allowing users to observe how RIS parameters are updated in real-time during the emulation. In practice, this interface can be replaced or extended by an external RIS controller.
\end{enumerate}

\section*{Directory Structure}
\dirtree{%
.1 /.
.2 Core/.
.3 Engine.py.
.2 Config/.
.3 Nodes.json.
.3 Output.json.
.3 RIS.json.
.3 Environment.json.
.2 External\_traffic/.
.2 Modules/.
.3 Channel\_functions.py.
.3 Mobility\_functions.py.
.3 Simulator\_functions.py.
}

\section*{Configuration Files}
\begin{itemize}
    \item \textbf{Nodes.json} – Defines nodes, IDs, locations, mobility, and operating frequencies.
    \item \textbf{Output.json} – Stores runtime IQ samples, node states, and logs.
    \item \textbf{RIS.json} – Defines RIS parameters and per-element responses.
    \item \textbf{Environment.json} – Defines simulation environment (dimensions, layout).
\end{itemize}

\section*{Interfaces}
The emulator provides SDR-like APIs:  

\begin{lstlisting}[language=Python]
Send_to_sim(Data, Operating_Frequency, Sample_Rate, Node_ID)
Receive_from_sim(Sample_Size, Operating_Frequency, Sample_Rate, Node_ID)
\end{lstlisting}

These mimic USRP-style functions such as:  
\begin{lstlisting}[language=Python]
usrp.send_waveform(samples,duration,center_freq,sample_rate,channel_ID,gain)
usrp.recv_num_samps(sample_size, center_freq, sample_rate, channel_ID, gain)
\end{lstlisting}

\section*{Running the Emulator}
\begin{enumerate}
    \item Define nodes and RIS parameters in \texttt{Config/Nodes.json} and \texttt{Config/RIS.json}.
    \item Start by issuing transmission/reception requests via \texttt{External\_traffic/usrp\_tx} and \texttt{usrp\_rx}.
    \item Run the engine:  
    \begin{lstlisting}[language=bash]
    python Core/Engine.py
    \end{lstlisting}
    \item Outputs (IQ samples, logs, state updates) are generated in \texttt{Config/Output.json} at each time step $\tau$.
\end{enumerate}

\section*{Example Usage}
Initial \texttt{Nodes.json} entry:
\begin{lstlisting}[language=json]
{
  "nodes": [
    {
      "id": "node_1",
      "location": [5, 2, 5],
      "mobility": {
        "type": "static",
        "speed": 0.0
      },
      "request": {},
      "current_mode": "idle",
      "fc": -1,
      "sample_rate": -1,
      "next_update": -1,
      "req_time": 0,
      "current_counter": -1,
      "data": []
    }
  ]
}
\end{lstlisting}

When a command is given from \texttt{usrp\_tx}, the node data and request is updated. Similarly, \texttt{usrp\_rx} requests trigger reception. Running \texttt{Engine.py} processes these requests and updates \texttt{Output.json} at each $\tau$. 

\noindent
\textbf{Note:} The emulation continues to run as long as the \texttt{current\_mode} of each node is set to \texttt{idle}. 
When a transmission or reception request is issued, the corresponding node state is updated accordingly, and 
the emulator processes the request until the node returns to the \texttt{idle} state. The file \texttt{Config/Output.json} is continuously updated during emulation. 
If the emulator is executed multiple times without clearing this file, new data will be appended 
to the existing contents. To avoid stacking results from previous runs, it is recommended to 
clear or reset \texttt{Output.json} before starting a new emulation session.


\section*{Extensibility}
\begin{itemize}
    \item Add new mobility models in \texttt{Modules/Mobility\_functions.py}.
    \item Implement custom channel models in \texttt{Modules/Channel\_functions.py}.
    \item Extend SDR emulation APIs in \texttt{External\_traffic/}.
\end{itemize}



\end{document}
