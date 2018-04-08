---
title: My Q&A on MPI (1)
date: 2018-04-05 00:07:55
tags:
---

**Q: Is MPI a language?**
A: No. It is just a library in C and Fortran to facilitate the message passing among multiple processors without shared memory.

**Q: I found there are many MPI libraries. Are they different or not?**
A: Yes, they are different. We call them different implementations of MPI, e.g. MPICH, MVAPICH2, Cray MPI, Intel MPI, SGI MPI, etc. MPI is just a standard of message passing interface. The standard only specifies some interfaces, e.g. function names, parameters, types, input/output, runtime behaviour… It gives some practical suggestions for the implementors. You can refer to this official document from MPI Forum for the latest MPI 3.0.

**Q: The most common errors of a parallel program is called ‘deadlock’, what is that?**
A: Deadlock means, at some stage, the processors are waiting for each other’s response. They just hang there forever. Normally it is due to incorrect arrangement of order of send and receive calls.
<!-- more -->
**Q: If two cores are running one program, both call MPI_Send(), will they cause deadlock?**
A: Most likely, but not sure. MPI_Send() is the standard blocking send mode. From the MPI Standard 3.4: “In this mode, it is up to MPI to decide whether outgoing messages will be buffered.” It depends on the runtime situations, e.g. data size (key factor),  network speed, buffer available space… etc. If MPI decides to buffer the outgoing messages, it is called “buffer send mode”; otherwise, it is called “synchronous send mode”. But as a MPI user, try to do not write codes which have some deadlock risks.

**Q: I am confused about “send call return”, “send-complete”, “message passing complete” and “local/non-local”. What’s the meaning?**
A: Firstly, we say there are two types of modes. One is blocking, the other is non-blocking. Blocking send (MPI_Send())’s return means the completion of the sending process, which means the outgoing message has been stored away. Here the “stored away” has two possibilities. One possibility is it is in the transit to the receiver (synchronous, non-local). Another possibility is it is stored into some system temporary memory (buffer, local). Only the first possibility means the message has been received by the receiver. 
For non-blocking mode (MPI_Isend()), it will always return immediately. This “return” only means the system gets the request from the sender to send messages.

**Q: The system will provides some temporary memory to store the messages automatically? Do our users need to worry about the memory leak?**
A: Yes. No. Those temporary memories are maintained by the MPI and system.

**Q: What’s the most important difference between blocking and non-blocking send?**
A: The “return” is the key. The return of blocking send means the user can modify the send buffer, but that of non-blocking send does not have that meaning. It must be accompanied with MPI_Wait or MPI_Test to obtain the send-completion.

**Q: I really do not want to let MPI to determine things. How can I use the buffer-send or synchronous-send directly?**
A: MPI_Bsend or MPI_Ssend. Especially for the Bsend, the user needs to prepare the memory by him/her-self and then provide the MPI with it for buffering purpose. Finally, the user is also responsible to release that bunch of memory to avoid any leak.

**Q: You mention “send-completion” and “send buffer is ready to modification”. Are they the same?**
A: Yes.

**Q: The message goes directly from the sender to receiver, right?**
A: No. In buffer mode, the message needs to be stored into some buffer first before going to the receiver. The buffer is either provided by the system automatically or by the user explicitly.

**Q: How big the buffer is? Is it equal to the size of the outgoing data?**
A: No. The size of the buffer is the size of outgoing data + overhead (MPI_BSEND_OVERHEAD). The overhead has the information about the message envelope. Normally, the MPI_BSEND_OVERHEAD is different in different implementations, platforms, versions. You can check it in mpi.h file. The unit is byte.

**Q: Can a processor send messages to itself?**
A: Yes, but not recommended to avoid any undefined behaviors.

**Q: If I am the receiver, after the MPI_Recv() returns, how can I double-check whether the message is correct or not.**
A: use the status obtained. Its type is MPI_Status, which contains MPI_SOURCE, MPI_TAG and MPI_Error for users to check.

**Q: Is Non-blocking necessarily asynchronous?**
A: No. The concept of blocking/non-blocking is totally different from the concept of a-synchronous.

**Q: For blocking send, which mode is the best?**
A: Hard to say, but generally MPI_Ssend(). Because it does not do any buffering definitely. Of course, the user needs to arrange the MPI_Recv() properly to minimize the overhead.

**Q: If my program is very large, which needs a lot of message passing. What’s the maximum tags supported by MPI?**
A: You can use the MPI_TAG_UB to find the maximum. From the Standard 3.0, that number is no less than 32767.

**Q: If the two processors belonging to two heterogeneous machines respectively, how can I make sure the data representation is correct in communication?**
A: The user should not worry about that except it communicating MPI_BYTE directly. From the MPI Standard, the integer, char will be maintained exactly. The floating number will be transformed into the closest one.

**Q: Would the synchronous send mode not buffer any data?**
A: No! This is a very interesting feature. The MPI implementation sometimes will make the decision for the user. I will use the MPI_Issend() to tell the difference. The code example is here:
{% codeblock lang:c%}
#include<iostream>
#include"mpi.h"
#include<vector>
#include<unistd.h>
using namespace std;
int main(int argc, char **argv)
{
  int rank, size, *sendbuf;
  MPI_Init(&argc, &argv);
  MPI_Comm_rank(MPI_COMM_WORLD, &rank);
  MPI_Comm_size(MPI_COMM_WORLD, &size);
  MPI_Request s_req, r_req;
  MPI_Status s_stat, r_stat;
  int ndata = 100000;
  vector<double> data;
  data.resize(ndata);
  if(rank==0){
    for(int i=0;i<ndata;i++){
      data[i] = i;
    }
    MPI_Issend(&data[0], ndata, MPI_DOUBLE, 1, 99, MPI_COMM_WORLD, &s_req);
    data[ndata-1]=9527;
    cout<<"The data from proc 0 before MPI_WAIT is:"<<endl;
    cout<<data[ndata-1]<<endl;
    MPI_Wait(&s_req, &s_stat);
    cout<<"The data from proc 0 after MPI_Wait is:"<<endl;
    cout<<data[ndata-1]<<endl;
    
  }else if(rank == 1){
    usleep(3e6);
    MPI_Recv(&data[0], ndata, MPI_DOUBLE, 0, 99, MPI_COMM_WORLD, &r_stat);
    cout<<"The data from proc 1 is:"<<endl;
    cout<<data[ndata-1]<<endl;
  }
  MPI_Barrier(MPI_COMM_WORLD);
  MPI_Finalize();
  return 0;
}
{% endcodeblock %}
{% raw %}
<p style="background-color: #000000; color: #00FF00">
The data from proc 0 before MPI_WAIT is:<br>
9527<br>
The data from proc 0 after MPI_Wait is:<br>
9527<br>
The data from proc 1 is:<br>
9527<br>
</p>
{% endraw %}
After change the ndata=10:
{% raw %}
<p style="background-color: #000000; color: #00FF00">
The data from proc 0 before MPI_WAIT is:<br>
9527<br>
The data from proc 0 after MPI_Wait is:<br>
9527<br>
The data from proc 1 is:<br>
9<br>
</p>
{% endraw %}
The outgoing messages were buffered into system temporary memory for small data (10 double variables) even if I use the MPI_Issend(). In this case, the modification on the send buffer will not change the data received by the receiver because it has already been buffered. For large data, MPI_Issend() determines to use synchrounous mode, so the modification will influcence the received data. At this point, I think MPI_Issend() is very similar with MPI_Isend(). (It does not strictly obey the MPI Standard?) Note: The above code is just for test purpose. Users always can not modify the send buffer before MPI_Wait().

**Q: Now, I change my mind. I want to use the buffer send, how to do that?**
A: See the code, especially the MPI_Pack_size, MPI_Buffer_attach and MPI_Buffer_detach. Look at the final data received by proc 1.
{% codeblock lang:c%}
#include<iostream>
#include"mpi.h"
#include<vector>
#include<unistd.h>
using namespace std;
int main(int argc, char **argv)
{
  int rank, size, *sendbuf;
  MPI_Init(&argc, &argv);
  MPI_Comm_rank(MPI_COMM_WORLD, &rank);
  MPI_Comm_size(MPI_COMM_WORLD, &size);

  MPI_Request s_req, r_req;
  MPI_Status s_stat, r_stat;
  int ndata = 10000;
  vector<double> data;
  data.resize(ndata);
  
  double* buffer;
  int bsize, tsize;
  vector<double> output;
  if(rank==0){
    for(int i=0;i<ndata;i++){
      data[i] = i;
    }
    
    MPI_Pack_size(ndata, MPI_DOUBLE, MPI_COMM_WORLD, &bsize);
    buffer = new double[bsize+MPI_BSEND_OVERHEAD];
    MPI_Buffer_attach(buffer, bsize+MPI_BSEND_OVERHEAD);
    MPI_Bsend(&data[0], ndata, MPI_DOUBLE, 1, 99, MPI_COMM_WORLD);
    cout<<"The proc 0 has finished buffering."<<endl;
    usleep(1000000);
    data[ndata-1]=9527;
    output.push_back(data[ndata-1]);
    MPI_Buffer_detach(&buffer, &tsize);
    cout<<"The buffer size is "<<tsize<<"."<<endl; 
    delete [] buffer;
  }else if(rank == 1){
    usleep(3e6);
    MPI_Recv(&data[0], ndata, MPI_DOUBLE, 0, 99, MPI_COMM_WORLD, &r_stat);
    output.push_back(data[ndata-1]);
  }
  MPI_Barrier(MPI_COMM_WORLD);
  for(int i=0;i<size;i++){
    if(i==rank){
      cout<<"The output from proc "<<i<<": "<<output[0]<<endl;
    }
    MPI_Barrier(MPI_COMM_WORLD);
  }
  MPI_Finalize();
  return 0;
}
{% endcodeblock %}
{% raw %}
<p style="background-color: #000000; color: #00FF00">
The proc 0 has finished buffering.<br>
The buffer size is 80096.<br>
The output from proc 0: 9527<br>
The output from proc 1: 9999<br>
</p>
{% endraw %}
The similar thing, the modification to the send buffer will not influence the data received by the receiver.
**Q: Is the MPI_Rsend() useful?**
A: Yes, but its usage should be only in some limited situations.

Q: Another question...
A: Today is too late. Let’s talk next time.

Reference:
[MPI Tutorial: http://mpitutorial.com/tutorials/](http://mpitutorial.com/tutorials/)
[MPI Standard 3.0 http://mpi-forum.org/docs/mpi-3.0/mpi30-report.pdf](http://mpi-forum.org/docs/mpi-3.0/mpi30-report.pdf)

