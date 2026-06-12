---
layout: page
title: Fortran & MPI Computing
description: High Performance Computing
img: assets/img/project1/project1thumbnail.png
importance: 1
category: work
---


<h2>Introduction</h2>
<hr class="bg-danger border-2 border-top border-primary-subtle">

Computer power is steadily increasing along with exabytes of data created each day. The computing power doubles every 2 years (Moore's Law), having an exponential rate of increase. However, Moore's Law cannot go on forever and one day we are going to reach a fundamental limit in which transistors approach the size of an atom. This technological singularity might be fully replaced or augmented one day with another technology (e.g., quantum computers, more complex multi-core chips). Meanwhile, major silicon companies are making technological breakthroughs to deal with this technlogical singularity as well as deal with othe issues such as clock speed, heat, memory bandwith limitations, etc. Having multicore systems have let to parallel computing to solve challenging problems such as computational chemistry, weather prediction, financial and economic modeling, protein folding, optimization, bioinformatics, climate modeling, and much more. 

The project described in this page was a final project I completed when taking an introductory course in High Performance Computing. I learned about parallel machine models and programming models, design principles such as partitioning, communication, agglomeration, mapping, and performance analysis. There are many programming languages, APIs, and parallel programming models for programming parallel computers. The project described here uses Fortran 90 and Message Passing Interface (MPI). Although Fortran has been around for many years and other programming languages have been recently created, Fortran is still used in high-performance computing for scientific and engineering research, including at <a href="https://www.nas.nasa.gov/publications/ams/2015/04-28-15.html"> NASA for mission-critical projects</a>. I hope this project inspires you to research, explore, and practice parallel computing in general.

<h2>Goals | Problem Statement</h2>
<hr class="bg-danger border-2 border-top border-primary-subtle">

<div class="card" style="width: auto;">
  <div class="card-header" style="text-align: center;">
    Given a 2D grid of cells randomly initialized with 0s and 1s
    to denote inactive and active states, respectively, change the state of a cell 
    given the following conditions
  </div>
  <ul class="list-group list-group-flush">
    <li class="list-group-item">1. If 3 neighboring cells are active and the current cell is active, then it remains active</li>
    <li class="list-group-item">2. If 3 neighboring cells are active and the current cell is inactive, then the inactive cell becomes active</li>
    <li class="list-group-item">3. If 2 neighbors are active, the current cell does not change</li>
    <li class="list-group-item">4. All other cases (e.g., more than 3 neighboring cells), the current cell remains inactive if it is already inactive, or becomes inactive if it is active</li>
    <li class="list-group-item">5. Assumme periodic boundary conditions</li>
  </ul>
</div>

<br>

<h2>Design Process</h2>
<hr class="bg-danger border-2 border-top border-primary-subtle">

In order to solve this problem, the Partition, Communication, Agglomeration, and Mapping (PCAM) process was executed as follows:

<div class="accordion" id="accordionPanelDesignProcess">
    <div class="accordion-item">
        <h2 class="accordion-header" id="panelsStayOpen-Partitionheading">
            <button class="accordion-button" type="button" data-bs-toggle="collapse" data-bs-target="#panelsStayOpen-collapsePartition" aria-expanded="true" aria-controls="panelsStayOpen-collapsePartition">
                Partition
            </button>
        </h2>
        <div id="panelsStayOpen-collapsePartition" class="accordion-collapse collapse show" aria-labelledby="panelsStayOpen-Partitionheading">
            <div class="accordion-body">
            For the 1D and 2D domain decomposition of the grid, the grid of active and inactive cells were decomposed in the x or the x-y directions. The general idea is to decompose the data and partition it along the rows or columns in such a way that the data is partitioned equally in order for the tasks to perform the same amount of computation. We can either map each fine-grained task to a 1x1 grid point for a total of (m x n) tasks as shown in Figure 1a, do a 1D domain decomposition along the columns, Figure 1b, or a 2D domain decomposition along the rows and columns, Figure 1c. In either of the latter cases, the data is partitioned equally so the tasks perform the same amount of computation.
            </div>
            <div class="row justify-content-sm-center">
                <div class="col-sm-8 mt-3 mt-md-0">
                    {% include figure.html path="assets/img/project1/grid.png" title="partition image" class="img-fluid rounded z-depth-1" %}
                    <div class="caption">
                        Figure 1: Different ways to partition a 4x4 grid. (a) Each 1x1 grid maps to a task, (b) 1D domain partitioning along the columns thus creating 2 (2x4) partitions of the same size, and (c) 2D domain partitioning along the columns and rows, thus creating 4 (2x2) partitions of the same size.
                    </div>
                </div>
            </div>
        </div>
    </div>
    <div class="accordion-item">
        <h2 class="accordion-header" id="panelsStayOpen-Communicationheading">
            <button class="accordion-button collapsed" type="button" data-bs-toggle="collapse" data-bs-target="#panelsStayOpen-collapseCommunication" aria-expanded="false" aria-controls="panelsStayOpen-collapseCommunication">
            Communication
            </button>
        </h2>
        <div id="panelsStayOpen-collapseCommunication" class="accordion-collapse collapse" aria-labelledby="panelsStayOpen-Communicationheading">
            <div class="accordion-body">
                In the communication phase of the design process, the goal is to have the tasks perform concurrent execution while avoiding unnecessary communication between the tasks. Therefore, we need to look at the 8 neighbors surrounding a single grid point. We need to look at the neighbors to the left, right, up, down and diagonals as shown in Figure 2b and 2c. In theory, a single grid point needs to communicate eight times with the surrounding neighbors in order to compute its state. For a simple matrix of size 4x4, Figure 2a, it would require 4x4x8 = 128 communications in total.
            </div>
            <div class="row justify-content-sm-center">
                <div class="col-sm-8 mt-3 mt-md-0">
                    {% include figure.html path="assets/img/project1/decomposition.png" title="communication image" class="img-fluid rounded z-depth-1" %}    
                    <div class="caption">
                        Figure 2: (a) A 4x4 matrix where each cell maps to a task. (b) Ghost cell of the original matrix a. If a 1x1 grid is a task, then this task needs all the information from the 8 neighbors. (c) Directions in which a 1x1 cell must look for the information. This 1x1 cell needs the information from all surrounding 8 neighbors.
                    </div>
                </div>
            </div>
        </div>
    </div>
    <div class="accordion-item">
        <h2 class="accordion-header" id="panelsStayOpen-Agglomerationheading">
            <button class="accordion-button collapsed" type="button" data-bs-toggle="collapse" data-bs-target="#panelsStayOpen-collapseAgglomeration" aria-expanded="false" aria-controls="panelsStayOpen-collapseAgglomeration">
            Agglomeration
            </button>
        </h2>
        <div id="panelsStayOpen-collapseAgglomeration" class="accordion-collapse collapse" aria-labelledby="panelsStayOpen-Agglomerationheading">
            <div class="accordion-body">
                In the agglomeration phase, we need to look a ways of reducing communication costs while maintaining scalability and flexibility. In the 1D decomposition case, if we partition the matrix into column partitions of the same size, Figure 3a, then we just need to send the data of the rightmost and leftmost columns to my neighbors (need to wrap around at the edges). In this case, the 4x4 grid is partitioned into 2x4 partitions, or 2 tasks, each task responsible for 8 grid points. Once I receive the information from my left and right neighbors, I can then use my local information and the information received to build a ghost cell as shown in Figure 3c. In this case, I just need to communicate to the neighbor to my left and to my right. In other words, I just need to send and receive information from my left and right neighbors and doing so I can account for the corners. This type of communication works for the 1D case and this can be verified with the ground truth shown in Figure 3b. 
                <br>
                <br>
                <div class="row justify-content-sm-center">
                    <div class="col-sm-8 mt-3 mt-md-0">
                        {% include figure.html path="assets/img/project1/agglomeration-1.png" title="agglomeration image" class="img-fluid rounded z-depth-1" %}    
                        <div class="caption">
                            Figure 3: (a) A 4x4 matrix with 2 (2x4) partitions. Each partition maps to a task. (b) Ghost cell of the original matrix a. This serves as the ground truth to verify that the communication between the tasks is correct. (c) Directions in which to transfer the information between tasks. A task just needs to send information to its left and right neighbors.
                        </div>
                    </div>
                </div>
                In the 2D case, we need to expand the 1D idea. I need some sort of structured communication in order to avoid too much communication. After looking at different solutions so as to decrease the total number of communications, I found that the best solution was to first send the data to the upper and lower neighbors as shown in Figure 4c. Once a partition received the information from its upper and lower neighbors, the local data and the data received can the be packaged and sent to the neighbors to the left and right as shown in Figure 4d. This takes care of providing the diagonal/corner information. In the end, a partition just needs to communicate a total of 4 times to send the information and 4 times to receive the information. This type of communication works for the 2D case and can be verified with the ground truth shown in Figure 4b.
                <br>
                <br>
                <div class="row justify-content-sm-center">
                    <div class="col-sm-8 mt-3 mt-md-0">
                        {% include figure.html path="assets/img/project1/agglomeration-2.png" title="agglomeration image" class="img-fluid rounded z-depth-1" %}    
                        <div class="caption">
                            Figure 4: (a) A 4x4 matrix with 4 (2x2) partitions. Each partition maps to a task for a total of 4 tasks. (b) Ghost cell of the original matrix a. This serves as the ground truth to verify that the communication between the tasks is correct. (c) Directions in which to initially transfer the information between tasks. A task first needs to send information to its upper and lower neighbors. (d) Once the information from the upper and lower neighbors has been received, a task must then send information to its left and right neighbors. After a task receives all the information from all 4 neighbors, then it can proceed to build a ghost cell and perform the computation.
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
    <div class="accordion-item">
        <h2 class="accordion-header" id="panelsStayOpen-Mappingheading">
            <button class="accordion-button collapsed" type="button" data-bs-toggle="collapse" data-bs-target="#panelsStayOpen-collapseMapping" aria-expanded="false" aria-controls="panelsStayOpen-collapseMapping">
            Mapping
            </button>
        </h2>
        <div id="panelsStayOpen-collapseMapping" class="accordion-collapse collapse" aria-labelledby="panelsStayOpen-Mappingheading">
            <div class="accordion-body">
                In the mapping phase, we need to specify where in the grid each task is to execute in order to minimize execution time. Since we are using domain decomposition and based on the observations during the agglomeration phase, we want to make sure that each task performs the same amount of computation and communication. In the 1D case, this can be accomplished by partitioning the grid vertically into partitions of equal column size as shown in Figure 5a for two different grid sizes. For the 2D case, this can be accomplished by partitioning the grid vertically and horizontally into partitions of equal column and row size as shown in Figure 5b. This ensures that all the tasks perform the same amount of communication.
                <br>
                <br>
                <div class="row justify-content-sm-center">
                    <div class="col-sm-8 mt-3 mt-md-0">
                        {% include figure.html path="assets/img/project1/mapping.png" title="mapping image" class="img-fluid rounded z-depth-1" %}    
                        <div class="caption">
                            Mapping task examples. (a) A 4x4 grid can be partitioned into 2 (2x4) partitions, where each partition corresponds to a task. (b) A 4x4 grid can be partitioned into 4 (2x2) partitions, where each partition corresponds to a task. The processor ranks are mapped in sequential order from left to right, top to bottom.
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>   
</div>
<br>
<h2>Algorithm Task Logic</h2>
<hr class="bg-danger border-2 border-top border-primary-subtle">
The algorithm presented here made used of the PCAM process outlined above. Each task is assigned to each processor and the task logic for the 1D decomposition case is shown in Algorithm listing 1. The task logic for the 2D decomposition case is shown in Algorithm listing 2. The basic idea of the design is to have a master node that distributes the data to the working nodes. Upon receiving the partition of the data that it needs to work on, the working node then transmits the edge information to its neighbors. Once a working node receives all the information it needs from its neighbors, then it builds the ghost cell and runs the game on its local data. Once it has completed all the computations, the working node then sends its results to the master node. Once the master node receives all the partitions from the workers, the game board is reconstructed and displayed on the screen. This process is executed for X number of iterations.
<div class="card" style="width: auto;">
    <div class="card-header" style="text-align: center;">
    Algorithm 1 - 1D domain decomposition task logic
    </div>
    <div class="card-header" style="text-align: left;">
        Input: a matrix of size (m x n), where m = n
        <br>
        Output: a matrix of size (m x n)
        <br>
        while (number of iterations to execute) {
            <br>
            <span class="tab">/*The master processor is always the processor with rank 0 */</span>
            <br>
            <span class="tab">if (master processor) { </span>
                <br>
                <span class="tab-2">initialize grid;</span>
                <br>
                <span class="tab-2">partition the grid and send a partition to a processor;</span>
                <br>
                <span class="tab-2">receive processed data from each processor;</span>
                <br>
                <span class="tab-2">reconstruct the grid;</span>
                <br>
            <span class="tab">} else { </span>
            <br>
                <span class="tab-2">/* The computation processors start from 1 to n */</span>
                <br>
            <span class="tab-2">receive the partition to process from master processor;</span>
            <br>
            <span class="tab-2">send data to left and right neighbors;</span>
            <br>
            <span class="tab-2">receive data from left and right neighbors;</span>
            <br>
            <span class="tab-2">build ghost cell;</span>
            <br>
            <span class="tab-2">perform computation;</span>
            <br>
            <span class="tab-2">send computed data back to master processor;</span>
            <br>
            <span class="tab">}</span>
            <br>
        }
    </div>
</div>
<br>
<div class="card" style="width: auto;">
    <div class="card-header" style="text-align: center;">
    Algorithm 2 - 2D domain decomposition task logic
    </div>
    <div class="card-header" style="text-align: left;">
        Input: a matrix of size (m x n), where m = n
        <br>
        Output: a matrix of size (m x n)
        <br>
        while (number of iterations to execute) {
            <br>
            <span class="tab">/*The master processor is always the processor with rank 0 */</span>
            <br>
            <span class="tab">if (master processor) { </span>
                <br>
                <span class="tab-2">initialize grid;</span>
                <br>
                <span class="tab-2">partition the grid and send a partition to a processor;</span>
                <br>
                <span class="tab-2">receive processed data from each processor;</span>
                <br>
                <span class="tab-2">reconstruct the grid;</span>
                <br>
            <span class="tab">} else { </span>
            <br>
                <span class="tab-2">/* The computation processors start from 1 to n */</span>
                <br>
            <span class="tab-2">receive the partition to process from the master processor;</span>
            <br>
            <span class="tab-2">send data to upper and lower neighbors;</span>
            <br>
            <span class="tab-2">receive data from upper and lower neighbors;</span>
            <br>
            <span class="tab-2">send data to left and right neighbors;</span>
            <br>
            <span class="tab-2">receive data from left and right neighbors;</span>
            <br>
            <span class="tab-2">build ghost cell;</span>
            <br>
            <span class="tab-2">perform computation;</span>
            <br>
            <span class="tab-2">send computed data back to master processor;</span>
            <br>
            <span class="tab">}</span>
            <br>
        }
    </div>
</div>
<br>
<br>
<h2>Source Code - Fortran</h2>
<hr class="bg-danger border-2 border-top border-primary-subtle">
The following listings show the source code for the sequential and 1D decomposition algorithms. Listing sequentialGoL.f90 shows the sequential implementation. Listing parallel1GoL.f90 shows the 1D parallel implementation.
<div class="accordion" id="accordionSourceCode">
    <div class="accordion-item">
        <h2 class="accordion-header">
            <button class="accordion-button collapsed" type="button" data-bs-toggle="collapse" data-bs-target="#collapseSeqAlg" aria-expanded="false" aria-controls="collapseSeqAlg">
            sequentialGoL.f90
            </button>
        </h2>
        <div id="collapseSeqAlg" class="accordion-collapse collapse" data-bs-parent="#accordionSourceCode">    
            <div class="accordion-body">
{% highlight fortran linenos %}
!---------------------------------------------------------------------------------
! sequentialGoL.f90
!
! Sequential algorithm used to check for correctness of parallel version
!
!	Assumptions: 
! 		- 1 = cell is active; 0 = cell is inactive
!		- The matrix is a square matrix
!
!	To run this program:
!		1. Modify the constants listed below 
!			- GAMEBOARD_SIZE, ITERATIONS, RANDOM_SEED
!		2. Execute the following commands:
!			> gfortran sequentialGoL.f90 -o seqgol
!			> ./seqgol
!
! Author: German H. Flores
!---------------------------------------------------------------------------------
PROGRAM sequentialGoL	
implicit none

!---------------------------------------------------------------------------------
! 						ONLY MODIFY THESE CONSTANTS
!---------------------------------------------------------------------------------
! Declare constants
!	- GAMEBOARD_SIZE: 	The size of the square matrix. This creates a
!						mxn matrix where m=n
!	- ITERATIONS: 		Total number of iterations to execute
!	- RANDOM_SEED:		The assignment of 1s and 0s to the board are
!						done randomly. This value is used to initialize
!						the pseudorandom number generator
!---------------------------------------------------------------------------------
integer, parameter :: GAMEBOARD_SIZE 	= 9
integer, parameter :: ITERATIONS 		= 1
integer, parameter :: RANDOM_SEED 		= 10 !TIME()
!---------------------------------------------------------------------------------
!						ONLY MODIFY THESE CONSTANTS
!---------------------------------------------------------------------------------


!---------------------------------------------------------------------------------
! Declare variables
!---------------------------------------------------------------------------------
integer :: m, n, gameBoardSize, iterationsCount
integer , allocatable :: gameBoard (:,:)
!---------------------------------------------------------------------------------


!---------------------------------------------------------------------------------
! Initialize variables
!---------------------------------------------------------------------------------
! Dynamically create a square matrix
!---------------------------------------------------------------------------------
gameBoardSize = GAMEBOARD_SIZE
m = gameBoardSize
n = gameBoardSize
allocate(gameBoard(m,n))

!Initialize matrix to 0 and total number of iterations to execute
gameBoard(:,:) = 0
iterationsCount = ITERATIONS
!---------------------------------------------------------------------------------


!---------------------------------------------------------------------------------
! MAIN CODE
!---------------------------------------------------------------------------------
! Display original matrix
!write(*,*) 'Empty gameboard:'
!call printMatrix(gameBoard,m,n)

! Display program configuration
call displayConfiguration()

! Initialize gameboard (1=alive, 0=dead)
call initializeGameBoard(gameBoard,m,n,RANDOM_SEED)

! Original matrix after assignment of 1s
write(*,*) 'Gameboard after initialization:'
call printMatrix(gameBoard,m,n)

! Run the game
call runGame(gameBoard, m, n, iterationsCount)
!---------------------------------------------------------------------------------


!---------------------------------------------------------------------------------
!Free memory
!---------------------------------------------------------------------------------
deallocate (gameBoard)
!---------------------------------------------------------------------------------


!---------------------------------------------------------------------------------
! Subroutines
!---------------------------------------------------------------------------------
CONTAINS

!---------------------------------------------------------------------------------
! Assuming periodic boundary conditions. Creates a ghost cell, which 
! contains the data plus the edges and corners
!---------------------------------------------------------------------------------
subroutine createGhostCell(cell, m, n, ghostCell)
	implicit none

	integer :: m, n, mNew, nNew
	integer, dimension(m,n) :: cell
	integer, dimension(m+2,n+2) :: ghostCell

	! Ghost cell has a total of 4 edges added around the cell
	mNew = m + 2
	nNew = n + 2

	! Copy inner elements
	ghostCell(2:mNew-1,2:nNew-1) = cell(:,:)

	! Copy corners
	ghostCell(1,1) = cell(m,n)
	ghostCell(1,nNew) = cell(m,1)
	ghostCell(mNew,1) = cell(1,n)
	ghostCell(mNew,nNew) = cell(1,1)

	! Copy side elements - wrap around
	ghostCell(2:mNew-1,1) = cell(:,n)
	ghostCell(2:mNew-1,nNew) = cell(:,1)
	ghostCell(1,2:nNew-1) = cell(m,:)
	ghostCell(nNew,2:nNew-1) = cell(1,:)

	return
end subroutine createGhostCell
!---------------------------------------------------------------------------------

!---------------------------------------------------------------------------------
! Run the game given the total number of iterations
!---------------------------------------------------------------------------------
subroutine runGame(gameBoard, m, n, iterations)
	implicit none

	integer :: i, j, iter, m, n, iterations
	integer :: mGhostCell, nGhostCell, sumTotal
	integer :: centerCell
	integer , dimension(m,n) :: gameBoard
	integer , allocatable :: ghostCell (:,:)
	integer , allocatable :: window (:,:)

	! Allocate window to look around eight neighbors
  	allocate(window(3,3))
  	allocate(ghostCell(m+2,n+2))
  	window(:,:) = 0
  	ghostCell(:,:) = 0

  	mGhostCell = m + 2
  	nGhostCell = n + 2
  	sumTotal = 0

	! Repeat iteration times
	do, iter=1,iterations

		! Create ghost cell and clear gameboard since it 
		! will be overwritten with the new results
		call createGhostCell(gameBoard, m, n, ghostCell)
		gameBoard(:,:) = 0

		! Navigate through the 2D map space
		do, i=2,mGhostCell-1
		 	do, j=2,nGhostCell-1

				! Create a window around each grid point
		  		window = ghostCell(i-1:i+1,j-1:j+1)
		  		centerCell = window(2,2) !Save indexed cell
		  		window(2,2) = 0 !Do not account for the element
		  			            !in the middle. Just look at the 
		  			            !8 neighbors
		  			            
		  		! Add the eight neighboring values
		    	sumTotal = sum(window)

		    	! If 3 neighbors are alive, cell will be alive 
		    	! (if already alive, remains alive; if was dead, 
		    	! becomes alive)
		   		if (sumTotal == 3) then
					gameBoard(i-1,j-1) = 1

				! If 2 neighbours are alive, no change in cell status.
				else if (sumTotal == 2) then
					gameBoard(i-1,j-1) = centerCell

				! All other cases, cell is dead (if was dead, remains 
				! dead; if was alive, becomes dead)
				else
					gameBoard(i-1,j-1) = 0
				end if
		    enddo
		enddo

		! Show board after each iteration
		write(*,*) 'Gameboard after iteration ', iter, ' :'
		call printMatrix(gameBoard,m,n)

		! Show total number of alive cells in the gameboard
		sumTotal = sum(gameBoard)
		write(*,*) 'Total number of alive cells: ', sumTotal
		write(*,"(A)") '-------------------------------------------'

	enddo

	!Free memory
	deallocate (window,ghostCell)

	return
end subroutine runGame
!---------------------------------------------------------------------------------

!---------------------------------------------------------------------------------
! Assigns random 1s to 50% of the matrix. Assumes that the board has
! been previously initialized with 0s
!---------------------------------------------------------------------------------
subroutine initializeGameBoard(mat, m, n, seed) 
	implicit none

	integer :: j, m, n
	integer :: mat(m,n)
	integer :: totalSize, seed

	totalSize = m*n

	!Randomly assign 1 to 50% of the cells
	call srand(seed)
	do j=1,ceiling(totalSize*0.50)
		mat(ceiling(rand()*m),ceiling(rand()*n)) = 1
	enddo

	return
end subroutine initializeGameBoard
!---------------------------------------------------------------------------------

!---------------------------------------------------------------------------------
! Prints a matrix of size mxn to the screen
!---------------------------------------------------------------------------------
subroutine printMatrix(mat,m,n)
	implicit none

	INTEGER :: i, m, n
  	INTEGER, DIMENSION(m,n) :: mat

  	write(*,"(A)") '-------------------------------------------'
  	do, i=1,m
    	write(*,*) (mat(i,:))
	enddo
	write(*,"(A)") '-------------------------------------------'
	write(*,"(A)") ''

	return
end subroutine printMatrix
!---------------------------------------------------------------------------------

!---------------------------------------------------------------------------------
! Show configuration of the program
!---------------------------------------------------------------------------------
subroutine displayConfiguration()
	implicit none

  	write(*,"(A)") '---------------------------------------------------'
  	write(*,"(A)") 'Program configuration: '
  	write(*,"(A)") '---------------------------------------------------'
	write(*,*) 'Game board size:', GAMEBOARD_SIZE, ' x ', GAMEBOARD_SIZE
	write(*,*) 'Number of iterations:', ITERATIONS
	write(*,*) 'Pseudorandom seed used:', RANDOM_SEED
	write(*,"(A)") '---------------------------------------------------'
	write(*,"(A)") ''

	return
end subroutine displayConfiguration
!---------------------------------------------------------------------------------

END PROGRAM sequentialGoL
{% endhighlight %}
            </div>
        </div>
    </div>
    <div class="accordion-item">
        <h2 class="accordion-header">
            <button class="accordion-button" type="button" data-bs-toggle="collapse" data-bs-target="#collapseOne" aria-expanded="true" aria-controls="collapseOne">
            parallel1DGoL.f90
            </button>
        </h2>
        <div id="collapseOne" class="accordion-collapse collapse" data-bs-parent="#accordionSourceCode">
            <div class="accordion-body">
{% highlight fortran linenos %}
!---------------------------------------------------------------------------------
! parallel1DGoL.f90
!
! Parallel 1D version
! 
!	Assumptions: 
! 		- 1 = cell is active; 0 = cell is inactive
!		- The matrix is a square matrix
!		- The size of the 1D partitions is the same. In other words, they 
!		  must fit evenly across the matrix
!		- The assignment of the processors is from left to right starting at 1.
!		  Therefore, 1 and numProcs-1 are at the boundary. Processor 0 is the
!		  master processor
!
!	Requirements:
!		- The number of processors set in parallel1DGoL.cmd file must be
!		  equal to be number of partitions + 1
!
!	To run this program:
!		1. Modify the constants listed below 
!			- GAMEBOARD_SIZE, ITERATIONS, PARTITON_SIZE, RANDOM_SEED
!
!		2. Execute with the following commands:
!			$ mpif90 parallel1DGoL.f90 -o par1Dgol
!			$ ./par1Dgol 
!
!   Program configuration examples:
!
! 		(GAMEBOARD_SIZE, PARTITION_SIZE, # OF PROCESSORS)
!		(     4        ,      2        ,       3        )
!		(     9        ,      3        ,       4        )
!		(     10       ,      2        ,       6        )
!		(     12       ,      3        ,       5        )
!		(     16       ,      4        ,       5        )
!
! Author: German H. Flores
!---------------------------------------------------------------------------------
PROGRAM parallel1DGoL
implicit none
include 'mpif.h'

!---------------------------------------------------------------------------------
! 						ONLY MODIFY THESE CONSTANTS
!---------------------------------------------------------------------------------
! Declare constants
!	- GAMEBOARD_SIZE: 	The size of the square matrix. This creates a
!						mxn matrix where m=n
!	- ITERATIONS: 		Total number of iterations to execute
!	- PARTITION_SIZE:	Total number of vertical partitions
!	- RANDOM_SEED:		The assignment of 1s and 0s to the board are
!						done randomly. This value is used to initialize
!						the pseudorandom number generator
!---------------------------------------------------------------------------------
integer, parameter :: GAMEBOARD_SIZE 	= 9
integer, parameter :: ITERATIONS 		= 1
integer, parameter :: PARTITION_SIZE 	= 3
integer, parameter :: RANDOM_SEED 		= 10 !TIME()
!---------------------------------------------------------------------------------
!						ONLY MODIFY THESE CONSTANTS
!---------------------------------------------------------------------------------


!---------------------------------------------------------------------------------
! Declare variables and other constants
!---------------------------------------------------------------------------------
integer, parameter :: LEFT = 0
integer, parameter :: RIGHT = 1

integer :: m, n, tag, count, iter
integer :: lowerLimit, upperLimit
integer :: procID
integer :: blockID, i
integer , allocatable :: gameBoard (:,:)
integer , allocatable :: ghostCell (:,:)
integer , allocatable :: boardSection (:,:)
integer , allocatable :: sectionResult (:,:)

integer , allocatable :: send_request_handles(:)
integer , allocatable :: recv_request_handles(:)
integer , allocatable :: send_request_stats(:,:)
integer , allocatable :: recv_request_stats(:,:)
integer :: recv_request_handle, send_request_handle
integer(kind=4), dimension(MPI_STATUS_SIZE,1) :: recv_request_stat
integer(kind=4), dimension(MPI_STATUS_SIZE,1) :: send_request_stat

integer myid, ierr, numprocs, errcode, masterProcessor
integer stat(MPI_STATUS_SIZE)

integer(kind=4), dimension(2) :: request_recv
integer(kind=4), dimension(2) :: request_send
integer(kind=4), dimension(MPI_STATUS_SIZE,2) :: stat_send
integer(kind=4), dimension(MPI_STATUS_SIZE,2) :: stat_recv

integer :: leftDestination, rightDestination

! Create custom BlockColumn derived type
! This is used to hold a column data to transfer to the
! neighbors. The id identifies the column as either left
! or right column. This is used to ensure the correct
! column in appended to the correct side of the neighbor
type BlockColumn
    sequence !force the derived type to be stored contiguously
    integer id
    integer(kind=4), dimension(GAMEBOARD_SIZE) :: bColumn
end type BlockColumn

! Create custom BlockSection derived type
! This is used to hold the section of the board that a given
! processor is to process. The id is the rank of the processor
! that processed the given section
type BlockSection
    sequence
    integer id
    integer(kind=4), dimension(GAMEBOARD_SIZE, PARTITION_SIZE) &
        :: bSection
end type BlockSection

! Derived types used to store the data sent/received by the 
! processors
type (BlockColumn) block_column_send
type (BlockColumn) block_column_recv
type (BlockColumn) block_columns_recv(2)

type (BlockSection) block_section_send
type (BlockSection) block_section_recv
type (BlockSection) block_sections_recv(PARTITION_SIZE)

integer blockColumnType, oldtypes(0:1), blockcounts(0:1)
integer offsets(0:1), extent
integer blockSectionType

call MPI_INIT(ierr)
    call MPI_COMM_RANK(MPI_COMM_WORLD, myid, ierr)
    call MPI_COMM_SIZE(MPI_COMM_WORLD, numprocs, ierr)

!--------------------------------------------------------------
! Initialize BlockColumn type
!--------------------------------------------------------------
! Setup description of the MPI_INTEGER ID 
offsets(0) = 0
oldtypes(0) = MPI_INTEGER
blockcounts(0) = 1

! Setup description of the 2 MPI_INTEGER fields
call MPI_TYPE_EXTENT(MPI_INTEGER, extent, ierr)
offsets(1) = 1 * extent
oldtypes(1) = MPI_INTEGER
blockcounts(1) = GAMEBOARD_SIZE

! Now define structured type and commit it 
call MPI_TYPE_STRUCT(2, blockcounts, offsets, oldtypes, & 
    blockColumnType, ierr)
call MPI_TYPE_COMMIT(blockColumnType, ierr)
!--------------------------------------------------------------
! Initialize BlockSection type
!--------------------------------------------------------------
! Setup description of the MPI_INTEGER ID 
offsets(0) = 0
oldtypes(0) = MPI_INTEGER
blockcounts(0) = 1

! Setup description of the 2 MPI_INTEGER fields
call MPI_TYPE_EXTENT(MPI_INTEGER, extent, ierr)
offsets(1) = 1 * extent
oldtypes(1) = MPI_INTEGER
blockcounts(1) = GAMEBOARD_SIZE*PARTITION_SIZE

! Now define structured type and commit it 
call MPI_TYPE_STRUCT(2, blockcounts, offsets, oldtypes, & 
    blockSectionType, ierr)
call MPI_TYPE_COMMIT(blockSectionType, ierr)
!--------------------------------------------------------------

! Initialize other variables and allocate memory
masterProcessor = 0
tag = 777
count = 1

! Dynamically allocate memory
m = GAMEBOARD_SIZE
n = GAMEBOARD_SIZE
allocate(gameBoard(m,n))
allocate(ghostCell(m+2,PARTITION_SIZE+2))
allocate(boardSection(m,PARTITION_SIZE))
allocate(sectionResult(m,PARTITION_SIZE))
allocate(send_request_handles(numprocs-1))
allocate(send_request_stats(MPI_STATUS_SIZE,numprocs-1))
allocate(recv_request_stats(MPI_STATUS_SIZE,numprocs-1))
allocate(recv_request_handles(numprocs-1))

!--------------------------------------------------------------
! 							MAIN CODE
!--------------------------------------------------------------
! Execute game X number of times as specified by the parameter
! "ITERATIONS"
do, iter=1,ITERATIONS

    !--------------------------------------------------------------
    ! The Master processor sends sections of the data to each
    ! processor, receives the processed data from each processor
    ! and reconstructs the gameboard again for the next
    ! iteration
    if (myid .eq. masterProcessor) then

        !--------------------------------------------------------------
        ! Only initialize gameboard in the first iteration
        !--------------------------------------------------------------
        if (iter .eq. 1) then
        
            ! Display program configuration
            call displayConfiguration()
        
            ! Initialize gameboard (1=alive, 0=dead)
            gameBoard(:,:) = 0
            call initializeGameBoard(gameBoard,m,n,RANDOM_SEED)
            write(*,*) 'Gameboard after initialization:'
            call printMatrix(gameBoard,m,n)
        endif
        !--------------------------------------------------------------



        !--------------------------------------------------------------
        ! Distribute the data to each processor
        !--------------------------------------------------------------
        ! Note:This could be improved by just using myid and the partition size
        ! 	to calculate the region of the gameboard that a processor is
        ! 	supposed to process. However, I make the master processor
        ! 	the node that controls all other processors and thus
        ! 	delegates the data to each one
        !--------------------------------------------------------------
        do, procID=1,numprocs-1

            lowerLimit = (procID*PARTITION_SIZE)-(PARTITION_SIZE-1)
            upperLimit = procID*PARTITION_SIZE

            ! int MPI_Isend(buffer, count, datatype, dest, tag,
            !     MPI_Comm comm, request )
            !
            ! buffer	- Address of send buffer
            ! count  	- Number of elements in send buffer
            ! datatype 	- MPI datatype of each send buffer element
            ! dest   	- Rank of destination
            ! tag    	- Message tag 
            ! comm   	- PI_Comm communicator
            ! request   - MPI_Request handle
            call MPI_ISEND(gameboard(:,lowerLimit:upperLimit), & 
                m*PARTITION_SIZE, MPI_INTEGER, procID, & 
                tag, MPI_COMM_WORLD, send_request_handles(procID), ierr)
        enddo

        call MPI_WAITALL (numprocs-1, send_request_handles,  &
            send_request_stats, ierr)
        !--------------------------------------------------------------


        !--------------------------------------------------------------
        ! Receive the processed data from each of the processors
        !--------------------------------------------------------------
        do, procID=1,numprocs-1
            call MPI_IRECV(block_sections_recv(procID), count,  & 
                blockSectionType, procID, tag, MPI_COMM_WORLD,  &
                recv_request_handles(procID), ierr)
        enddo

        call MPI_WAITALL (numprocs-1, recv_request_handles, & 
            recv_request_stats, ierr)


        !!# write(*,*) 'Received Result from processor ', & 
        !block_sections_recv(1)%id, ' :'
        !!# call printMatrix(block_sections_recv(1)%bSection, & 
        !GAMEBOARD_SIZE,PARTITION_SIZE)
        !--------------------------------------------------------------


        !--------------------------------------------------------------
        ! Reconstruct the gameboard with the new data from the processors
        !--------------------------------------------------------------
        do, i=1, numprocs-1
            blockID = block_sections_recv(i)%id
            lowerLimit = (blockID*PARTITION_SIZE)-(PARTITION_SIZE-1)
            upperLimit = blockID*PARTITION_SIZE
            gameboard(:,lowerLimit:upperLimit) = block_sections_recv(i)%bSection
        enddo

        write(*,*) 'The gameboard after iteration ', iter, 'is: '
        call printMatrix(gameBoard,m,n)
        !--------------------------------------------------------------

    !--------------------------------------------------------------
    ! All other processors receive the data from the master processor,
    ! send/receive data to/from the left and right neighbors, creates
    ! a ghost cell, processes the data, and send the processed data
    ! back to the master processor
    !--------------------------------------------------------------
    else
        !--------------------------------------------------------------
        ! Receive the data to process from the master processor
        ! int MPI_Irecv(buffer, count, datatype, source,
        !				tag, comm, request)
        !
        ! buffer 	- Address of receive buffer
        ! count 	- Number of elements in receive buffer
        ! datatype 	- MPI_Datatype of each receive buffer element
        ! source 	- Rank of source
        ! tag 		- Message tag
        ! comm 		- MPI_Comm communicator
        ! request   - MPI_Request handle
        call MPI_IRECV(boardSection, m*PARTITION_SIZE, MPI_INTEGER, & 
            masterProcessor, tag, MPI_COMM_WORLD, recv_request_handle, ierr)

        call MPI_WAITALL (1, recv_request_handle, recv_request_stat, ierr)

        !write(*,*) 'Data received from master node by ', myid, ' :'
        !call printMatrix(boardSection,m,PARTITION_SIZE)
        !--------------------------------------------------------------



        !--------------------------------------------------------------
        ! Set correct left and right destination ranks, either neighbors
        ! to the left and right or wrap around
        ! If left or right most boundary, the data needs to be wrapped 
        ! around
        !--------------------------------------------------------------
        ! Since assignment of processors is from left to right,
        ! processor with rank = 1 is the leftmost processor; processor
        ! with rank = numprocs-1 is the rightmost processor
        if (myid .eq. 1) then

            leftDestination = numprocs-1
            rightDestination = myid+1

        ! If rightmost boundary, the data needs to be wrapped around
        else if (myid .eq. numprocs-1) then

            leftDestination = myid-1
            rightDestination = 1

        ! Send data to left and right neighbors
        else
            leftDestination = myid-1
            rightDestination = myid+1
        endif
        !--------------------------------------------------------------



        !--------------------------------------------------------------
        ! Send left and right columns to the left or right neighbors
        ! Wrap around if at the edges
        !--------------------------------------------------------------
        ! Copy the leftmost column and send to its left neighbor (numprocs-1)
        block_column_send = BlockColumn(LEFT,boardSection(:,1))
        call MPI_ISEND(block_column_send, count, blockColumnType,  & 
            leftDestination, tag, MPI_COMM_WORLD, request_send(1), ierr)

        ! Copy the rightmost column and send to its right neighbor
        block_column_send = BlockColumn(RIGHT,boardSection(:,PARTITION_SIZE))
        call MPI_ISEND(block_column_send, count, blockColumnType, & 
            rightDestination, tag, MPI_COMM_WORLD, request_send(2), ierr)

        ! Wait until all the data has been sent
        call MPI_WAITALL (2, request_send, stat_send, ierr)
        !--------------------------------------------------------------



        !--------------------------------------------------------------
        ! Receive left and right data from the neighbors
        !--------------------------------------------------------------
        ! Receive the column data from the neighbors
        call MPI_IRECV(block_columns_recv(1), count, blockColumnType, & 
            leftDestination, tag, MPI_COMM_WORLD, request_recv(1), ierr)

        call MPI_IRECV(block_columns_recv(2), count, blockColumnType, & 
            rightDestination, tag, MPI_COMM_WORLD, request_recv(2), ierr)

        ! Wait until all the data has been received
        call MPI_WAITALL (2, request_recv, stat_recv, ierr)

        !write(*,*)"processor ",myid," with cellblock ID ", &
        !block_columns_recv(1)%id, " has data: ",block_columns_recv(1)%bColumn(:)

        !write(*,*)"processor ",myid," with cellblock ID", &
        !block_columns_recv(2)%id," has data: ",block_columns_recv(2)%bColumn(:)
        !--------------------------------------------------------------



        !--------------------------------------------------------------
        ! Build ghost cell and run the game
        !--------------------------------------------------------------
        ! Once the columns from the left and right have been received, build
        ! ghost cell and run the game on the data
        call buildGhostCell(block_columns_recv, ghostCell, GAMEBOARD_SIZE, & 
            PARTITION_SIZE, boardSection)
        !write(*,*)"The ghost cell is for processor : ", myid
        !call printMatrix(ghostCell,GAMEBOARD_SIZE+2,PARTITION_SIZE+2)

        call runGame(ghostCell, GAMEBOARD_SIZE+2, PARTITION_SIZE+2, & 
            sectionResult, myid)

        !write(*,*)"The section result is for processor : ", myid
        !call printMatrix(sectionResult,GAMEBOARD_SIZE,PARTITION_SIZE)
        !--------------------------------------------------------------



        !--------------------------------------------------------------
        ! Send results back to the master node
        !--------------------------------------------------------------
        block_section_send = BlockSection(myid,sectionResult)
        call MPI_ISEND(block_section_send, count, blockSectionType, &
            masterProcessor, tag, MPI_COMM_WORLD, send_request_handle, ierr)

        call MPI_WAITALL (1, send_request_handle, send_request_stat, ierr)
        !--------------------------------------------------------------

    endif

enddo
!--------------------------------------------------------------
! 						END OF MAIN CODE
!--------------------------------------------------------------

! Free memory
deallocate (gameBoard, ghostCell, boardSection, sectionResult)
deallocate (send_request_handles, send_request_stats)
deallocate (recv_request_stats, recv_request_handles)

call MPI_FINALIZE(ierr)
    stop
    
!---------------------------------------------------------------------------------
! Subroutines
!---------------------------------------------------------------------------------
CONTAINS

!---------------------------------------------------------------------------------
! Build ghost cell
! Assuming periodic boundary conditions
!---------------------------------------------------------------------------------
subroutine buildGhostCell(blockColumns, ghostCell, m, n, boardDataSection)
    implicit none

    ! Declare variables and allocate memory
    integer :: m, n, mNew, nNew
    type (BlockColumn) blockColumns(2)
    integer, dimension(m+2,n+2) :: ghostCell
    integer, dimension(m,n) :: boardDataSection
    integer , allocatable :: leftColumn(:)
    integer , allocatable :: rightColumn(:)

    allocate(leftColumn(GAMEBOARD_SIZE))
    allocate(rightColumn(GAMEBOARD_SIZE))

    ! Ghost cell has a total of 4 edges added around the partition
    mNew = m + 2
    nNew = n + 2

    ! Identify left and right sides
    ! This might not be necessary, but doing so ensures the column
    ! is appended on the correct side since using asynchronous send/recv
    if (blockColumns(1)%id .eq. LEFT) then
        rightColumn = blockColumns(1)%bColumn
        leftColumn = blockColumns(2)%bColumn
    else
        rightColumn = blockColumns(2)%bColumn
        leftColumn = blockColumns(1)%bColumn
    endif

    ! Copy inner elements
    ghostCell(2:mNew-1,2:nNew-1) = boardDataSection(:,:)

    ! Copy corners
    ghostCell(1,1) = leftColumn(m)
    ghostCell(mNew,1) = leftColumn(1)
    ghostCell(1,nNew) = rightColumn(m)
    ghostCell(mNew,nNew) = rightColumn(1)

    ! Copy side elements - wrap around
    ghostCell(2:mNew-1,1) = leftColumn
    ghostCell(2:mNew-1,nNew) = rightColumn

    ! Copy upper and lower elements
    ghostCell(1,2:nNew-1) = boardDataSection(m,:)
    ghostCell(mNew,2:nNew-1) = boardDataSection(1,:)

    ! Free memory
    deallocate (leftColumn, rightColumn)

    return
end subroutine buildGhostCell
!---------------------------------------------------------------------------------


!---------------------------------------------------------------------------------
! Run the game on the ghostCell
!---------------------------------------------------------------------------------
subroutine runGame(ghostCell, mGhostCell, nGhostCell, sectionResult, processorID)
    implicit none
    
    ! Declare variables and allocate memory
    integer :: i, j, m, n
    integer :: mGhostCell, nGhostCell, sumTotal
    integer :: centerCell, processorID
    integer , dimension(mGhostCell,nGhostCell) :: ghostCell
    integer , dimension(mGhostCell-2,nGhostCell-2) :: sectionResult
    integer , allocatable :: window (:,:)

    ! Allocate window to look around eight neighbors
    allocate(window(3,3))
    window(:,:) = 0
    sectionResult(:,:) = 0
    sumTotal = 0

    ! Navigate through the 2D map space
    do, i=2,mGhostCell-1
        do, j=2,nGhostCell-1

            ! Create a window around each grid point
            window = ghostCell(i-1:i+1,j-1:j+1)
            centerCell = window(2,2) !Save indexed cell
            window(2,2) = 0 !Do not account for the element
                            !in the middle. Just look at the 
                                !8 neighbors
            
            ! Add the eight neighboring values
            sumTotal = sum(window)

            ! If 3 neighbours are alive, cell will be alive 
            ! (if already alive, remains alive; if was dead, 
            ! becomes alive)
            if (sumTotal == 3) then
                sectionResult(i-1,j-1) = 1

            ! If 2 neighbours are alive, no change in cell status.
            else if (sumTotal == 2) then
                sectionResult(i-1,j-1) = centerCell

            ! All other cases, cell is dead (if was dead, remains 
            ! dead; if was alive, becomes dead)
            else
                sectionResult(i-1,j-1) = 0
            end if
        enddo
    enddo

    ! Show total number of alive cells in the gameboard
    sumTotal = sum(sectionResult)
    write(*,"(A)") '---------------------------------------------------'
    write(*,*) 'Total number of alive cells from processor ', & 
                processorID, ' is: ', sumTotal
    write(*,"(A)") '---------------------------------------------------'

    !Free memory
    deallocate (window)

    return
end subroutine runGame
!---------------------------------------------------------------------------------


!---------------------------------------------------------------------------------
! Assigns random 1s to 50% of the matrix
!---------------------------------------------------------------------------------
subroutine initializeGameBoard(mat, m, n, seed) 
    implicit none

    integer :: j, m, n
    integer :: mat(m,n)
    integer :: totalSize, seed

    totalSize = m*n

    !Randomly assign 1 to 50% of the cells
    call srand(seed)
    do j=1,ceiling(totalSize*0.50)
        mat(ceiling(rand()*m),ceiling(rand()*n)) = 1
    enddo

    return
end subroutine initializeGameBoard
!---------------------------------------------------------------------------------


!---------------------------------------------------------------------------------
! Prints a matrix of size mxn to the screen
!---------------------------------------------------------------------------------
subroutine printMatrix(mat,m,n)
    implicit none

    INTEGER :: i, m, n
    INTEGER, DIMENSION(m,n) :: mat

    write(*,"(A)") '---------------------------------------------------'
    do, i=1,m
        write(*,*) (mat(i,:))
    enddo
    write(*,"(A)") '---------------------------------------------------'
    write(*,"(A)") ''

    return
end subroutine printMatrix
!---------------------------------------------------------------------------------


!---------------------------------------------------------------------------------
! Show configuration of the program
!---------------------------------------------------------------------------------
subroutine displayConfiguration()
    implicit none

    write(*,"(A)") '---------------------------------------------------'
    write(*,"(A)") 'Program configuration: '
    write(*,"(A)") '---------------------------------------------------'
    write(*,*) 'Game board size:', GAMEBOARD_SIZE, ' x ', GAMEBOARD_SIZE
    write(*,*) 'Partition size:', PARTITION_SIZE, ' x ', GAMEBOARD_SIZE
    write(*,*) 'Number of iterations:', ITERATIONS
    write(*,*) 'Number of required processors:', GAMEBOARD_SIZE/ &
                                                PARTITION_SIZE + 1
    write(*,*) 'Pseudorandom seed used:', RANDOM_SEED
    write(*,"(A)") '---------------------------------------------------'
    write(*,"(A)") ''

    return
end subroutine displayConfiguration
!---------------------------------------------------------------------------------

END PROGRAM parallel1DGoL
{% endhighlight %}
            </div>
        </div>
    </div>
</div>
<br>
<h2>Conclusion</h2>
<hr class="bg-danger border-2 border-top border-primary-subtle">
This project shows how to design and implement parallel algorithms using the PCAM process, Fortran, and MPI. I hope this inspires you to learn more about Fortran and experiment creating parallel programming projects. If you are interested in seeing the source code for parallel2DGoL.f90, or you have any questions, please send me an e-mail.
<br>
<h2>References</h2>
<hr class="bg-danger border-2 border-top border-primary-subtle">
[1] <a href="https://www.fortran90.org">Fortran 90</a>
<br>
[2] <a href="https://www.open-mpi.org">Open MPI</a>