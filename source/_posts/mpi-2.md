---
title: My Q&A on MPI (2)
date: 2018-04-07 11:58:17
tags:
---

**Q: What did we talk about last time? Could you do a brief summary?**
A: Last time, we mentioned about the 4 send modes (standard, synchronous, buffered and ready). We also tested the MPI_Issend() and found it is not always synchronous although the function name contains a 's' representing it should be synchronous. Some concepts are clarified, e.g. 'local/nonlocal send-completion', 'send-return'. The non-blocking is mentioned a little bit.

**Q: Good. I think if I can design the code very well, I do not need the non-blocking communication, right?**
A: Yes, theoretically you can, but it's very hard for complex communication context. To avoid many design issues or deadlock, I prefer the non-blocking operations, not only for the simpler design, but sometimes, the efficiency is much better than the blocking counterparts.

**Q: I know the non-blocking send is MPI_Isend(), right?**
A: Right, but not complete. In fact, the non-blocking send also has 4 modes (standard, synchronous, buffered and ready). Last time, in fact, I talked about the non-blocking synchronous send -- MPI_Issend(). They are very similar. The difference from blocking send is MPI_I*send() will return immediately.

**Q: Are the blocking and non-blocking send and receive compatible with each other?**
A: Yes. You can use blocking send to send the data out and the data can be received by a non-blocking receive, and vice versa.
<!-- more -->
**Q: In syntax, what's the most evident difference between the MPI_Send() and MPI_Isend()?**
A: Here I put the MPI Standard 3.0 here:
{% codeblock lang:c%}
// Blocking send
int MPI_Send(const void *buf, int count, MPI_Datatype datatype, int dest, int tag, MPI_Comm comm)
// Non-blocking send
int MPI_Isend(const void *buf, int count, MPI_Datatype datatype, int dest, int tag, MPI_Comm comm, MPI_Request *request)
{% endcodeblock %}
We can see there is an extra argument of MPI_Isend(), it's type is MPI_Request. 
**Q: What's the MPI_Request type?**
A: The MPI_Request is an type for non-blocking communication. It contains some information about the communication associated with this request. Unlike the blocking operator, once it's returned, the send-buffer is released and ready to be reused. The return of non-blocking operator means nothing, user needs some thing (here the request handle) to track the status of the communication. There is a function named "MPI_Request_get_status()" to let the user to check the status through the request object.

**Q: You say the MPI_Request is like a tracker. It means the some variables inside this object wil change with the different communication stages. I think the default/initial value is important because it must represent some default/initial status, right?**
A: Good question. In fact, the default value of MPI_Request is not defined. In my computer, it is 0. It is better to set it to be MPI_REQUEST_NULL if necessary. It is a constant number defined in mpi.h file. In the mpich 3.0.2 version, it is
{% codeblock lang:c%}
//line 318: 
#define MPI_REQUEST_NULL   ((MPI_Request)0x2c000000)
{% endcodeblock %}
After the tracking is finished, the object needs to be reset(freed) to be MPI_REQUEST_NULL by system or the user.

**Q: What will happen if I track a MPI_Request object with value of MPI_REQUEST_NULL?**
A: The monitoring flag will be true, which means the request is complete. It sounds weired because how can the tracker report 'successful' with a empty request. In fact, it is very useful in some complex situations to make the code more symmetric. You will discover this feature in the future by yourself.

**Q: How to release/free the MPI_Request object?**
A: If you use MPI_Wait(), it will be freed by the system automatically. If you use MPI_Test(), unless the communication is finished (flag=true), the request will not be freed. If user wants to free it, just use MPI_Request_free(). Here is the code:
{% codeblock lang:c%}
#include "mpi.h"
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <iostream>
using namespace std;
int main(int argc, char **argv)
{
    double a[10];
    MPI_Status status;
    MPI_Request s_req;
    int got;
    
    int rank, size;
    MPI_Init(&argc, &argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    
    if(rank==0){
        
        for(int j=0;j<10;j++){
            a[j] = j*0.1+2.13;
        }
        
        cout<<"The constant MPI_REQUEST_NULL is "<<MPI_REQUEST_NULL<<endl;
        
        MPI_Issend(a, 10, MPI_DOUBLE, 1,0,MPI_COMM_WORLD, &s_req);
        
        MPI_Request_free(&s_req);
        
        if(s_req == MPI_REQUEST_NULL){
            cout<<"The s_req is MPI_REQUEST_NULL"<<endl;
        }else{
            cout<<"The s_req is not MPI_REQUEST_NULL"<<endl;
        }
        
        do{
            MPI_Test(&s_req,&got,&status);
            cout<<"got is "<<got<<endl;
        }while(!got);
        
        if(s_req == MPI_REQUEST_NULL){
            cout<<"The s_req is MPI_REQUEST_NULL"<<endl;
        }else{
            cout<<"The s_req is not MPI_REQUEST_NULL"<<endl;
        }
    }else if(rank==1){
        usleep(2e6);
        MPI_Recv(a, 10, MPI_DOUBLE, 0,0,MPI_COMM_WORLD, &status);
        for(int i=0;i<10;i++){
            cout<<"a["<<i<<"]="<<a[i]<<endl;
        }
    }

    MPI_Finalize();
    return 0;
}
{% endcodeblock %}
The output is
{% raw %}
<p style="background-color: #000000; color: #00FF00">
The constant MPI_REQUEST_NULL is 738197504<br>
The s_req is MPI_REQUEST_NULL<br>
got is 1<br>
The s_req is MPI_REQUEST_NULL<br>
a[0]=2.13<br>
a[1]=2.23<br>
a[2]=2.33<br>
a[3]=2.43<br>
a[4]=2.53<br>
a[5]=2.63<br>
a[6]=2.73<br>
a[7]=2.83<br>
a[8]=2.93<br>
a[9]=3.03<br>
</p>
{% endraw %}
We can notice that even the request object is freed immediately after the MPI_Issend() before the receiver starts to receive data, the communication will not fail. The 'tracker' will not influence the communication itself.

**Q: The non-blocking mode is much flexible than the blocking mode. Now I want to gather some data from N processors, I need to call (N-1) times MPI_Irecv()?**
A: No. MPI has the collective communication functions. Here I will only put one which I think the most practical and a little complex. It is MPI_Gatherv():
{% codeblock lang:c%}
#include "mpi.h"
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <iostream>
using namespace std;
int main(int argc, char **argv)
{
    int position, i;
    double send[3];
    double recv[6];
    MPI_Status status;
    int disp[3]={0,1,3};
    int count[3]={1,2,3};
    int rank, size;
    MPI_Init(&argc, &argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    int nsend;
    if(rank == 0){
        send[0]=1.0;
        nsend=1;
    }else if(rank == 1){
        send[0]=2.0;
        send[1]=3.0;
        nsend=2;
    }else if(rank == 2){
        send[0]=4.0;
        send[1]=5.0;
        send[2]=6.0;
        nsend=3;
    }
    MPI_Gatherv(send, nsend,MPI_DOUBLE, recv,count,disp,MPI_DOUBLE,0,MPI_COMM_WORLD);
    if(rank==0){
        for(int i=0;i<6;i++){
            cout<<"recv["<<i<<"]="<<recv[i]<<endl;
        }
    }
    
    MPI_Finalize();
    return 0;
}
{% endcodeblock %}
{% raw %}
<p style="background-color: #000000; color: #00FF00">
recv[0]=1<br>
recv[1]=2<br>
recv[2]=3<br>
recv[3]=4<br>
recv[4]=5<br>
recv[5]=6<br>
</p>
{% endraw %}
We can see the proc 0 is the root. Proc 0,1,2 will send different sized data to the root. Each procs prepare the pointer of the send buffer as well as the data count. The 'recv', 'count', 'disp' are only meaningful for the root processor. The 'count' and 'disp' are both arrays.

**Q: Nice, the collective communication saves me much energy. Does it have any drawbacks?**
A: It is not flexible. Every processors in the MPI_COMM_WORLD must execute this operator. If you only want the data from the processors with odd ranks, you can not use it directly. Of course, you can always use the send/recv to communicate, but if we want to use the collective communication, we should first put those odd ranks together (maybe create a new group or set to store those ranks) and then call thie collective communication within this group. This capability is also a part of MPI Standard, which will be talked about next time.

**Q: Up to now, all the data for communication is just the basic datatypes, e.g. integer, float number, character..., could we communicate the user-defined structure directly between processors?**
A: The short answer is NO. Any information must be transformed into several MPI datatypes before communication. However, MPI provides the user a better way to 'pack' the user's own structured data. They are MPI_Pack() and MPI_Unpack(). Each pack can have different types of data. It is not used intensively, here I give the code directly.
{% codeblock lang:c%}
#include "mpi.h"
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <iostream>


using namespace std;
int main(int argc, char **argv)
{
    int position, i;
    double a[10];
    char buff[1000];
    MPI_Status status;
    int rank, size;
    MPI_Init(&argc, &argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    if(rank==0){
        
        for(int j=0;j<10;j++){
            a[j] = j*0.1+2.13;
        }
        
        int len[2];
        MPI_Aint disp[2];
        MPI_Datatype type[2], newtype;
        i=9527;
        len[0] = 1;
        len[1] = 10;
        MPI_Address(&i, disp);
        MPI_Address(a,  disp+1);
        cout<<"Proc 0: the address of i is "<<disp[0]<<endl;
        cout<<"Proc 0: the address of a is "<<disp[1]<<endl;
        type[0] = MPI_INT;
        type[1] = MPI_DOUBLE;
        MPI_Type_struct(2, len, disp, type, &newtype);
        MPI_Type_commit(&newtype);
        
        int psize = 0;
        MPI_Pack_size(1, newtype, MPI_COMM_WORLD, &psize);
        position = 0;
        // 1st way: pack the new type
        //MPI_Pack(MPI_BOTTOM, 1, newtype, buff, psize, &position, MPI_COMM_WORLD);
        // 2nd way: directly pack the old types
        MPI_Pack(&i, 1, MPI_INT, buff, psize, &position, MPI_COMM_WORLD);
        cout<<"Proc 0: position after 1st MPI_Pack is "<<position<<endl;
        MPI_Pack(a, 10, MPI_DOUBLE, buff, psize, &position, MPI_COMM_WORLD);
        cout<<"Proc 0: position after 2nd MPI_Pack is "<<position<<endl;
        MPI_Send(buff, psize, MPI_PACKED, 1,0,MPI_COMM_WORLD);
        MPI_Type_free(&newtype);
    }else if(rank==1){
        
        MPI_Recv(buff, 1000, MPI_PACKED, 0,0,MPI_COMM_WORLD, &status);
        // 1000 is the size in byte
        position = 0;
        MPI_Unpack(buff,1000,&position,&i,1,MPI_INT,MPI_COMM_WORLD);
        
        cout<<"Proc 1: After first unpack, the position is "<<position<<endl;
        cout<<"Proc 1: The value i is "<<i<<endl;
        
        MPI_Unpack(buff,1000,&position,a,10,MPI_DOUBLE, MPI_COMM_WORLD);
        
        cout<<"Proc 1: After second unpack, the position is "<<position<<endl;
        
        for(int j=0;j<10;j++){
            cout<<"Proc 1: a["<<j<<"]="<<a[j]<<endl;
        }
    }
    MPI_Finalize();
    return 0;
}
{% endcodeblock %}
{% raw %}
<p style="background-color: #000000; color: #00FF00">
Proc 0: the address of i is 140726724892152<br>
Proc 0: the address of a is 140726724892240<br>
Proc 0: position after 1st MPI_Pack is 4<br>
Proc 0: position after 2nd MPI_Pack is 84<br>
Proc 1: After first unpack, the position is 4<br>
Proc 1: The value i is 9527<br>
Proc 1: After second unpack, the position is 84<br>
Proc 1: a[0]=2.13<br>
Proc 1: a[1]=2.23<br>
Proc 1: a[2]=2.33<br>
Proc 1: a[3]=2.43<br>
Proc 1: a[4]=2.53<br>
Proc 1: a[5]=2.63<br>
Proc 1: a[6]=2.73<br>
Proc 1: a[7]=2.83<br>
Proc 1: a[8]=2.93<br>
Proc 1: a[9]=3.03<br>
</p>
{% endraw %}

**Q/A: We are actors, see you next time.**

Reference:
[MPI Tutorial: http://mpitutorial.com/tutorials/](http://mpitutorial.com/tutorials/)
[MPI Standard 3.0 http://mpi-forum.org/docs/mpi-3.0/mpi30-report.pdf](http://mpi-forum.org/docs/mpi-3.0/mpi30-report.pdf)