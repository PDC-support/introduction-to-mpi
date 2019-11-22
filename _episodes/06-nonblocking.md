---
title: "Non-blocking Communication"
teaching: 10
exercises: 20
questions:
- "How do I interleave communication and computation?"
objectives:
- "Introduce `MPI_Isend`, `MPI_Irecv`, `MPI_Test` and `MPI_Wait`."
keypoints:
- "Non-blocking functions allows interleaving communication and computation."
---

## Non-Blocking Communication

### Send and Receive

In one of the previous lessons we used the `MPI_Send` and `MPI_Recv` functions
to communicate between the ranks.
We saw that these functions are blocking:
`MPI_Send` will only return when the program can safely modify the send buffer and
`MPI_Recv` will only return once the data has been received and
written to the receive buffer.
This is safe and usually straightforward, but causes the program to wait
while the communication is happening.
Usually there is computation that we could perform while waiting for data.

The MPI standard includes non-blocking versions of the send and receive functions,
`MPI_Isend` and `MPI_Irecv`.
These function will return immediately, giving you more control of the flow
of the program. After calling them, it is not safe to modify the sending or
the receiving buffer, but the program is free to continue with other operations.
When it needs the data in the buffers, it needs to make sure the communication process
is complete using the `MPI_Wait` and `MPI_Test` functions.

> ## `MPI_Isend`
>
>~~~
> int MPI_Isend(
>    void* data,
>    int count,
>    MPI_Datatype datatype,
>    int destination,
>    int tag,
>    MPI_Comm communicator,
>    MPI_Request* request)
>~~~
>
> | `data`:         | Pointer to the start of the data being sent |
> | `count`:        | Number of elements to send |
> | `datatype`:     | The type of the data being sent |
> | `destination`:  | The rank number of the rank the data will be sent to |
> | `tag`:          | A message tag (integer) |
> | `communicator`: | The communicator (we have used MPI_COMM_WORLD in earlier examples) |
> | `request`:      | Pointer for writing the request structure |
{: .callout .show-c}

> ## `MPI_Irecv`
>~~~
> int MPI_Irecv(
>    void* data,
>    int count,
>    MPI_Datatype datatype,
>    int source,
>    int tag,
>    MPI_Comm communicator,
>    MPI_Request* request)
>~~~
>
> | `data`:         | Pointer to where the received data should be written |
> | `count`:        | Maximum number of elements received |
> | `datatype`:     | The type of the data being received |
> | `source`:       | The rank number of the rank sending the data |
> | `tag`:          | A message tag (integer) |
> | `communicator`: | The communicator (we have used MPI_COMM_WORLD in earlier examples) |
> | `request`:      | Pointer for writing the request structure |
>
{: .callout .show-c}

> ## `MPI_Isend`
>
>~~~
> MPI_Isend(BUF, COUNT, DATATYPE, DEST, TAG, COMM, REQUEST, IERROR)
>    <type>    BUF(*)
>    INTEGER    COUNT, DATATYPE, DEST, TAG, COMM, REQUEST, IERROR
>~~~
>
> | `BUF`:      | Vector containing the data to send |
> | `COUNT`:    | Number of elements to send |
> | `DATATYPE`: | The type of the data being sent |
> | `DEST`:     | The rank number of the rank the data will be sent to |
> | `TAG`:      | A message tag (integer) |
> | `COMM`:     | The communicator (we have used MPI_COMM_WORLD in earlier examples) |
> | `REQUEST`:  | Request handle |
> | `IERROR`:   | Error status |
>
{: .callout .show-fortran}

> ## `MPI_Irecv`
>~~~
> MPI_Irecv(BUF, COUNT, DATATYPE, SOURCE, TAG, COMM, REQUEST, IERROR)
>    <type>    BUF(*)
>    INTEGER    COUNT, DATATYPE, SOURCE, TAG, COMM,
>    INTEGER    REQUEST, IERROR
>~~~
>
> | `BUF`:      | Vector the received data should be written to             |
> | `COUNT`:    | Maximum number of elements received                       |
> | `DATATYPE`: | The type of the data being received                       |
> | `SOURCE`:   | The rank number of the rank sending the data              |
> | `TAG`:      | A message tag (integer)                                   |
> | `COMM`:     | The communicator (we have used MPI_COMM_WORLD in earlier examples) |
> | `REQUEST`:  | Request handle                                            |
> | `IERROR`:   | Error status |
>
{: .callout .show-fortran}

> ## `MPI.Comm.isend`
>
>~~~
> def isend(self, obj, int dest, int tag=0)
>~~~
>
> | `obj`:          | The Python object being sent |
> | `dest`:         | The rank number of the rank the data will be sent to |
> | `tag`:          | A message tag (integer) |
>
{: .callout .show-python}

> ## `MPI.Comm.irecv`
>~~~
> def irecv(self, buf=None, int source=ANY_SOURCE, int tag=ANY_TAG)
>~~~
>
> | `buf`:          | The buffer object to where the received data should be written |
> | `source`:       | The rank number of the rank sending the data |
> | `tag`:          | A message tag (integer) |
>
{: .callout .show-python}

There's one new parameter here, a request.
This is used to keep track of each separate transfer started by the program.
You can use it to check the status of a transfer using the `MPI_Test` function,
or call `MPI_Wait` to wait until the transfer is complete.


## Test and Wait

`MPI_Test` will return the status of the transfer specified by a request and
`MPI_Wait` will wait until the transfer is complete before returning.
The request can be created by either `MPI_Isend` or `MPI_Irecv`.

> ## `MPI_Test`
>
>~~~
> int MPI_Test(
>    MPI_Request* request,
>    int * flag,
>    MPI_Status* status)
>~~~
>
> | `request`:      | The request |
> | `flag`:         | Pointer for writing the result of the test |
> | `status`:       | A pointer for writing the exit status of the MPI command |
{: .callout .show-c}

> ## `MPI_Wait`
>~~~
> int MPI_Wait(
>    MPI_Request* request,
>    MPI_Status* status)
>~~~
>
> | `request`:      | The request |
> | `status`:       | A pointer for writing the exit status of the MPI command |
>
{: .callout .show-c}

> ## `MPI_Test`
>
>~~~
> MPI_TEST(REQUEST, FLAG, STATUS, IERROR)
>    LOGICAL    FLAG
>    INTEGER    REQUEST, STATUS(MPI_STATUS_SIZE), IERROR
>~~~
>
> | `REQUEST`:  | The request |
> | `FLAG`:     | Pointer for writing the result of the test |
> | `STATUS`:   | A pointer for writing the exit status of the MPI command |
> | `IERROR`:   | Error status |
{: .callout .show-fortran}

> ## `MPI_Wait`
>~~~
>MPI_WAIT(REQUEST, STATUS, IERROR)
>    INTEGER    REQUEST, STATUS(MPI_STATUS_SIZE), IERROR
>~~~
>
> | `REQUEST`:  | The request |
> | `STATUS`:   | A pointer for writing the exit status of the MPI command |
> | `IERROR`:   | Error status |
>
{: .callout .show-fortran}

> ## `MPI.Request.test`
>
>~~~
> def test(self, Status status=None)
>~~~
>
> | `status`:       | A pointer for writing the exit status of the MPI command |
>
{: .callout .show-python}

> ## `MPI.Request.wait`
>~~~
> def wait(self, Status status=None)
>~~~
>
> | `status`:       | A pointer for writing the exit status of the MPI command |
>
{: .callout .show-python}


### Examples

These functions can be used similarly to `MPI_Send` and `MPI_Recv`.
Here is how you could replace `MPI_Send` and `MPI_Recv` in the program that
sends the "Hello World!" string by `MPI_ISend`, `MPI_IRecv` and `MPI_Wait`:

~~~
#include <stdio.h>
#include <mpi.h>

int main(int argc, char** argv) {
  int rank, n_ranks;
  int my_first, my_last;
  int numbers = 10;
  MPI_Request request;

  // First call MPI_Init
  MPI_Init(&argc, &argv);

  // Check that there are at least two ranks
  MPI_Comm_size(MPI_COMM_WORLD,&n_ranks);
  if( n_ranks < 2 ){
    printf("This example requires at least two ranks");
    MPI_Finalize();
    return(1);
  }

  // Get my rank
  MPI_Comm_rank(MPI_COMM_WORLD,&rank);

  if( rank == 0 ){
     char *message = "Hello, world!\n";
     MPI_Isend(message, 16, MPI_CHAR, 1, 0, MPI_COMM_WORLD, &request);
  }

  if( rank == 1 ){
     char message[16];
     MPI_Status status;
     MPI_Irecv(message, 16, MPI_CHAR, 0, 0, MPI_COMM_WORLD, &request);
     MPI_Wait( &request, &status );
     printf("%s",message);
  }

  // Call finalize at the end
  return MPI_Finalize();
}
~~~
{: .source .language-c .show-c}


~~~
program hello

    implicit none
    include "mpif.h"

    integer rank, n_ranks, request, ierr
    integer status(MPI_STATUS_SIZE)
    character(len=13)  message

    ! First call MPI_Init
    call MPI_Init(ierr)

    ! Check that there are at least two ranks
    call MPI_Comm_size(MPI_COMM_WORLD, n_ranks, ierr)
    if (n_ranks < 2) then
        write(6,*) "This example requires at least two ranks"
        error stop
    end if

    ! Get my rank
    call MPI_Comm_rank(MPI_COMM_WORLD, rank, ierr)

    if (rank == 0) then
        message = "Hello, world!"
        call MPI_Isend( message, 13, MPI_CHARACTER, 1, 0, MPI_COMM_WORLD, request, ierr )
    end if

    if (rank == 1) then
        call MPI_Irecv( message, 13, MPI_CHARACTER, 0, 0, MPI_COMM_WORLD, request, ierr )
        call MPI_WAIT( request, status, ierr )
        write(6,*) message
    end if

    ! Call MPI_Finalize at the end
    call MPI_Finalize(ierr)
end
~~~
{: .source .language-fortran .show-fortran}

~~~
from mpi4py import MPI
import sys

numbers = 10;

# Check that there are at least two ranks
n_ranks = MPI.COMM_WORLD.Get_size()
if n_ranks < 2:
    print("This example requires at least two ranks")
    sys.exit(1)

# Get my rank
rank = MPI.COMM_WORLD.Get_rank()

if rank == 0:
    message = "Hello, world!"
    req = MPI.COMM_WORLD.isend(message, dest=1, tag=0)

if rank == 1:
    req = MPI.COMM_WORLD.irecv(source=0, tag=0)
    message = req.wait()
    print(message)
~~~
{: .source .language-python .show-python}


> ## Non-Blocking Communication
>
> Here is the blocking example again.
> Fix the problem using `MPI_Isend`, `MPI_Irecv` and `MPI_Wait`.
>
> If you encounter a segmentation fault,
> think about whether a buffers have been
> released by MPI before you free them.
>
> ~~~
> #include <stdio.h>
> #include <stdlib.h>
> #include <mpi.h>
>
> int main(int argc, char** argv) {
>    int rank, n_ranks, neighbour;
>    int n_numbers = 10000;
>    int *send_message;
>    int *recv_message;
>    MPI_Status status;
>
>    send_message = malloc(n_numbers*sizeof(int));
>    recv_message = malloc(n_numbers*sizeof(int));
>
>    // First call MPI_Init
>    MPI_Init(&argc, &argv);
>
>    // Get my rank and the number of ranks
>    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
>    MPI_Comm_size(MPI_COMM_WORLD, &n_ranks);
>
>    // Check that there are exactly two ranks
>    if( n_ranks != 2 ){
>         printf("This example requires exactly two ranks\n");
>         MPI_Finalize();
>         return(1);
>    }
>
>    // Call the other rank the neighbour
>    if( rank == 0 ){
>       neighbour = 1;      
>    } else {
>       neighbour = 0;
>    }
>
>    // Generate numbers to send
>    for( int i=0; i<n_numbers; i++){
>       send_message[i] = i;
>    }
>
>    // Send the message to other rank
>    MPI_Send(send_message, n_numbers, MPI_INT, neighbour, 0, MPI_COMM_WORLD);
>
>    // Receive the message from the other rank
>    MPI_Recv(recv_message, n_numbers, MPI_INT, neighbour, 0, MPI_COMM_WORLD, &status);
>    printf("Message received by rank %d \n", rank);
>
>    // Call finalize at the end
>    free(send_message);
>    free(recv_message);
>
>    return MPI_Finalize();
> }
> ~~~
> {: .source .language-c .show-c}
>
>
>~~~
>program hello
>
>    implicit none
>    include "mpif.h"
>     
>    integer, parameter :: n_numbers=10000
>    integer i
>    integer rank, n_ranks, neighbour, ierr
>    integer status(MPI_STATUS_SIZE)
>    integer send_message(n_numbers)
>    integer recv_message(n_numbers)
>
>    ! First call MPI_Init
>    call MPI_Init(ierr)
>
>    ! Get my rank and the number of ranks
>    call MPI_Comm_rank(MPI_COMM_WORLD, rank, ierr)
>    call MPI_Comm_size(MPI_COMM_WORLD, n_ranks, ierr)
>
>    ! Check that there are exactly two ranks
>    if (n_ranks .NE. 2) then
>         write(6,*) "This example requires exactly two ranks"
>         error stop
>    end if
>
>    ! Call the other rank the neighbour
>    if (rank == 0) then
>        neighbour = 1
>    else
>        neighbour = 0
>    end if
>
>    ! Generate numbers to send
>    do i = 1, n_numbers
>        send_message(i) = i;
>    end do
>
>    ! Send the message to other rank
>    call MPI_Send( send_message, n_numbers, MPI_INTEGER, neighbour, 0, MPI_COMM_WORLD, ierr )
>
>    ! Receive the message from the other rank
>    call MPI_Recv( recv_message, n_numbers, MPI_INTEGER, neighbour, 0, MPI_COMM_WORLD, status, ierr )
>    write(6,*) "Message received by rank", rank
>
>    ! Call MPI_Finalize at the end
>    call MPI_Finalize(ierr)
>end
>~~~
>{: .source .language-fortran .show-fortran}
>
> ~~~
> from mpi4py import MPI
> import sys
>
> n_numbers = 10000
>
> # Get my rank and the number of ranks
> rank = MPI.COMM_WORLD.Get_rank()
> n_ranks = MPI.COMM_WORLD.Get_size()
>
> # Check that there are exactly two ranks
> if n_ranks != 2:
>     print("This example requires exactly two ranks")
>     sys.exit(1)
>
> # Call the other rank the neighbour
> if rank == 0:
>     neighbour = 1
> else:
>     neighbour = 0
>
> # Generate numbers to send
> send_message = []
> for i in range(n_numbers):
>     send_message.append(i)
>
> # Send the message to other rank
> MPI.COMM_WORLD.send(send_message, dest=neighbour, tag=0)
>
> # Receive the message from the other rank
> recv_message = MPI.COMM_WORLD.recv(source=neighbour, tag=0)
> print("Message received by rank", rank)
> ~~~
>{: .source .language-python .show-python}
>
>
>> ## Solution
>>
>> ~~~
>> #include <stdio.h>
>> #include <stdlib.h>
>> #include <mpi.h>
>>
>> int main(int argc, char** argv) {
>>    int rank, n_ranks, neighbour;
>>    int n_numbers = 10000;
>>    int *send_message;
>>    int *recv_message;
>>    MPI_Status status;
>>    MPI_Request request;
>>    int return_value;
>>
>>    send_message = malloc(n_numbers*sizeof(int));
>>    recv_message = malloc(n_numbers*sizeof(int));
>>
>>    // First call MPI_Init
>>    MPI_Init(&argc, &argv);
>>
>>    // Get my rank and the number of ranks
>>    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
>>    MPI_Comm_size(MPI_COMM_WORLD, &n_ranks);
>>
>>    // Call the other rank the neighbour
>>    if( rank == 0 ){
>>       neighbour = 1;      
>>    } else {
>>       neighbour = 0;
>>    }
>>
>>    // Generate numbers to send
>>    for( int i=0; i<n_numbers; i++){
>>       send_message[i] = i;
>>    }
>>
>>    // Send the message to other rank
>>    MPI_Isend(send_message, n_numbers, MPI_INT, neighbour, 0, MPI_COMM_WORLD, &request);
>>
>>    // Receive the message from the other rank
>>    MPI_Irecv(recv_message, n_numbers, MPI_INT, neighbour, 0, MPI_COMM_WORLD, &request);
>>    MPI_Wait( &request, &status );
>>    printf("Message received by rank %d \n", rank);
>>
>>    // Call finalize before freeing messages
>>    return_value = MPI_Finalize();
>>
>>    free(send_message);
>>    free(recv_message);
>>    return return_value;
>> }
>> ~~~
>>{: .source .language-c}
>{: .solution .show-c}
>
>
>> ## Solution
>>
>> ~~~
>>program hello
>>
>>   implicit none
>>   include "mpif.h"
>>    
>>   integer, parameter :: n_numbers=10000
>>   integer i
>>   integer rank, n_ranks, neighbour, request, ierr
>>   integer status(MPI_STATUS_SIZE)
>>   integer send_message(n_numbers)
>>   integer recv_message(n_numbers)
>>
>>   ! First call MPI_Init
>>   call MPI_Init(ierr)
>>
>>   ! Get my rank and the number of ranks
>>   call MPI_Comm_rank(MPI_COMM_WORLD, rank, ierr)
>>   call MPI_Comm_size(MPI_COMM_WORLD, n_ranks, ierr)
>>
>>   ! Check that there are exactly two ranks
>>   if (n_ranks .NE. 2) then
>>        write(6,*) "This example requires exactly two ranks"
>>        error stop
>>   end if
>>
>>   ! Call the other rank the neighbour
>>   if (rank == 0) then
>>       neighbour = 1
>>   else
>>       neighbour = 0
>>   end if
>>
>>   ! Generate numbers to send
>>   do i = 1, n_numbers
>>       send_message(i) = i;
>>   end do
>>
>>   ! Send the message to other rank
>>   call MPI_Isend( send_message, n_numbers, MPI_INTEGER, neighbour, 0, MPI_COMM_WORLD, request, ierr )
>>
>>   ! Receive the message from the other rank
>>   call MPI_Irecv( recv_message, n_numbers, MPI_INTEGER, neighbour, 0, MPI_COMM_WORLD, request, ierr )
>>   call MPI_WAIT( request, status, ierr )
>>   write(6,*) "Message received by rank", rank
>>
>>   ! Call MPI_Finalize at the end
>>   call MPI_Finalize(ierr)
>>end
>> ~~~
>>{: .source .language-fortran}
>{: .solution .show-fortran}
>
>
>> ## Solution
>>
>> ~~~
>> from mpi4py import MPI
>> import sys
>>
>> n_numbers = 10000
>>
>> # Get my rank and the number of ranks
>> rank = MPI.COMM_WORLD.Get_rank()
>> n_ranks = MPI.COMM_WORLD.Get_size()
>>
>> # Check that there are exactly two ranks
>> if n_ranks != 2:
>>     print("This example requires exactly two ranks")
>>     sys.exit(1)
>>
>> # Call the other rank the neighbour
>> if rank == 0:
>>     neighbour = 1
>> else:
>>     neighbour = 0
>>
>> # Generate numbers to send
>> send_message = []
>> for i in range(n_numbers):
>>     send_message.append(i)
>>
>> # Send the message to other rank
>> req = MPI.COMM_WORLD.isend(send_message, dest=neighbour, tag=0)
>>
>> # Receive the message from the other rank
>> req = MPI.COMM_WORLD.irecv(source=neighbour, tag=0)
>> recv_message = req.wait()
>> print("Message received by rank", rank)
>> ~~~
>>{: .source .language-python}
>{: .solution .show-python}
>
{: .challenge}



{% include links.md %}
