---
layout: post
title: Diagrams as Code
---


1. Diagrams (python-based)
2. MermaidJS (node-based)


	### portproxy in windows
	netsh interface portproxy add v4tov4 listenport=80 connectaddress=ip-of-server-on-internet connectport=23 listenaddress=ip-of-windows-machine protocol=tcp

	### remove portproxy
	netsh interface portproxy delete v4tov4 listenport=8081 listenaddress=0.0.0.0 protocol=tcp
