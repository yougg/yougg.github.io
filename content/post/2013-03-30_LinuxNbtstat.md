---
Categories: []
Description: ""
Tags: []
title: "nbtstat Linux版, 通过IP获取主机名"
date: 2013-03-30T00:00:00+08:00
draft: false
---

```c
/* NETBIOS name lookup tool - by eSDee of Netric (www.netric.org)
 * yeh.. i was really bored :-)
 *
 * [esdee@pant0ffel] gcc -o nbtstat nbtstat.c && ./nbtstat 10.0.0.4
 * Request sent, waiting for reply... [ctrl-c to abort]
 * Name                Type
 * ----------------------------------
 * HOEPELKO-ESPU0B <00> UNIQUE Workstation Service
 * HOEPELKO-ESPU0B <20> UNIQUE File Server Service
 * WORKGROUP       <00> GROUP  Domain Name
 * WORKGROUP       <1e> GROUP  Browser Service Elections
 * HOEPELKO-ESPU0B <03> UNIQUE Messenger Service
 * ADMINISTRATOR   <03> UNIQUE Messenger Service
 * WORKGROUP       <1d> UNIQUE Master Browser
 * ..__MSBROWSE__. <01> GROUP  Master Browser
 * MAC-addres = 00-02-a5-e1-fd-b7
 * [/home/esdee/nbtstat] */

#include  <stdio.h>
#include  <string.h>
#include  <sys/types.h>
#include  <sys/socket.h>
#include  <netinet/in.h>
#include  <netdb.h>
#include  <unistd.h>

char nbtname[]= /* netbios name packet */
{
	0x80,0xf0,0x00,0x10,0x00,0x01,0x00,0x00,
	0x00,0x00,0x00,0x00,0x20,0x43,0x4b,0x41,
	0x41,0x41,0x41,0x41,0x41,0x41,0x41,0x41,
	0x41,0x41,0x41,0x41,0x41,0x41,0x41,0x41,
	0x41,0x41,0x41,0x41,0x41,0x41,0x41,0x41,
	0x41,0x41,0x41,0x41,0x41,0x00,0x00,0x21,
	0x00,0x01
};

int display();

int main(int argc, char *argv[])
{
	char temp[16];

	unsigned char recv[256];
	unsigned char *ptr;

	struct sockaddr_in server;
	struct hostent *hp;

	int s;
	int total;

	unsigned int nb_num;
	unsigned int nb_type;

	int i = 0;

	if (argc < 2) {
		fprintf(stderr, "Usage: %s <hostname>\n", argv[0]);
		return 1;
	}

	if ((hp = gethostbyname(argv[1])) == NULL)  {
		fprintf(stderr,"Error: Unable to resolve %s\n", argv[1]);
		return 1;
	}

	if ((s = socket(PF_INET, SOCK_DGRAM, 17)) < 0)  { /* 17 = UDP */
		perror("socket");
		return 1;
	}

	memset(recv,0x0, sizeof(recv));

	bzero((char *) &server, sizeof(server));
	bcopy(hp->h_addr, (char *) &server.sin_addr, hp->h_length);

	server.sin_family = hp->h_addrtype;
	server.sin_port = htons(137); /* netbios-ns */

	if (sendto(s, nbtname, sizeof(nbtname), 0, (struct sockaddr *) &server, sizeof(server)) < 0)  {
		perror("sendto");
		return 1;
	} else {
		fprintf(stdout, "Request sent, waiting for reply... [ctrl-c to abort]\n");

		read(s, recv, sizeof(recv) - 1);

		fprintf(stdout,"Name                Type\n"
					   "----------------------------------\n");
		ptr=recv+57;
		 total=*(ptr - 1); /* max names */

		while(ptr < recv + sizeof(recv)) {
			memset(temp,0x0, sizeof(temp));
			strncpy(temp, ptr, 15);         /* copies the name into temp */

			ptr+=15;
			nb_num  = *ptr;
			nb_type = *(ptr + 1);
			ptr+=3;

			if (i==total) {          /* max names reached */
				ptr-=19;         /* sets the pointer to the mac_addres field */
				fprintf(stdout,"\nMAC-addres = %02x-%02x-%02x-%02x-%02x-%02x\n\n",
						   *(ptr + 1), *(ptr + 2), *(ptr + 3),
						   *(ptr + 4), *(ptr + 5), *(ptr + 6));
				break;
			}

			display(temp,nb_num,nb_type);
			i++;
		}
	}
	close(s);
	return  0;
}

int display(char *name, unsigned int number, unsigned int type)
{
	char description[256];
	int i;

	memset(description, 0x0, sizeof(description));

	/* list taken from http://support.microsoft.com/default.aspx?scid=KB;EN-US;q163409& */
	/* 0x04 - UNIQUE */
	/* 0x80 - GROUP */

	switch(number) {
		case 0x00:
			if (type <= 0x80) {
					strncpy(description, "UNIQUE Workstation Service", sizeof(description) - 1);
			} else {
					strncpy(description, "GROUP  Domain Name", sizeof(description) - 1);
			}
			break;
		case 0x01:
			if (type <= 0x80) {
					strncpy(description, "UNIQUE Messenger Service", sizeof(description) - 1);
			} else {
					strncpy(description, "GROUP  Master Browser", sizeof(description) - 1);
			}
			break;
		case 0x03:
			strncpy(description, "UNIQUE Messenger Service", sizeof(description) - 1);
			break;
		case 0x06:
			strncpy(description, "UNIQUE RAS Server Service", sizeof(description) - 1);
			break;
		case 0x1b:
			strncpy(description, "UNIQUE Domain Master Browser", sizeof(description) - 1);
			break;
		case 0x1c:
			strncpy(description, "GROUP  Domain Controllers", sizeof(description) - 1);
			break;
		case 0x1d:
			strncpy(description, "UNIQUE Master Browser", sizeof(description) - 1);
			break;
		case 0x1e:
			if (type >= 0x80) strncpy(description, "GROUP  Browser Service Elections", sizeof(description) - 1);
			break;
		case 0x1F:
			strncpy(description, "UNIQUE NetDDE Service", sizeof(description) - 1);
			break;
		case 0x20:
			strncpy(description, "UNIQUE File Server Service", sizeof(description) - 1);
			break;
		case 0x21:
			strncpy(description, "UNIQUE RAS Client Service", sizeof(description) - 1);
			break;
		case 0x22:
			strncpy(description, "UNIQUE Microsoft Exchange Interchange(MSMail Connector)", sizeof(description) - 1);
			break;
		case 0x23:
			strncpy(description, "UNIQUE Microsoft Exchange Store", sizeof(description) - 1);
			break;
		case 0x24:
			strncpy(description, "UNIQUE Microsoft Exchange Directory", sizeof(description) - 1);
			break;
		case 0x30:
			strncpy(description, "UNIQUE Modem Sharing Server Service", sizeof(description) - 1);
			break;
		case 0x31:
			strncpy(description, "UNIQUE Modem Sharing Client Service", sizeof(description) - 1);
			break;
		case 0x42:
			strncpy(description, "UNIQUE Mcaffee Anti-Virus", sizeof(description) - 1);
			break;
		case 0x43:
			strncpy(description, "UNIQUE SMS Clients Remote Control", sizeof(description) - 1);
			break;
		case 0x44:
			strncpy(description, "UNIQUE SMS Administrators Remote Control Tool", sizeof(description) - 1);
			break;
		case 0x45:
			strncpy(description, "UNIQUE SMS Clients Remote Chat", sizeof(description) - 1);
			break;
		case 0x46:
			strncpy(description, "UNIQUE SMS Clients Remote Transfer", sizeof(description) - 1);
			break;
		case 0x4C:
			strncpy(description, "UNIQUE DEC Pathworks TCPIP service on Windows NT", sizeof(description) - 1);
			break;
		case 0x52:
			strncpy(description, "UNIQUE DEC Pathworks TCPIP service on Windows NT", sizeof(description) - 1);
			break;
		case 0x6a:
			strncpy(description, "UNIQUE Microsoft Exchange IMC", sizeof(description) - 1);
			break;
		case 0x87:
			strncpy(description, "UNIQUE Microsoft Exchange MTA", sizeof(description) - 1);
			break;
		case 0xbe:
			strncpy(description, "UNIQUE Network Monitor Agent", sizeof(description) - 1);
			break;
		case 0xbf:
			strncpy(description, "UNIQUE Network Monitor Application", sizeof(description) - 1);
			break;
		default:
			strncpy(description, "UNIQUE Unknown", sizeof(description) - 1);
			break;
	}

	for (i=0; i < strlen(name); i++) /* replaces weird chars with dots */
		if (name[i] < 31 || name[i] > 126) name[i] = '.';

	if (name) fprintf(stdout, "%s <%02x> %s\n", name, number, description);
	return 0;

}
```
