---
title: 2D numerical integration
layout: post
---

Here we illustrate the use of a matrix inversion based 2D numerical integration, which is a path-independent procedure, to recover a 2D matrix from partially known elements (i.e., the boundary conditions) and the gradients along the vertical and horizontal directions.

### The ground truth data
The $$4\times4$$ ground truth matrix ($$\mathbf{M}$$) to be reconstructed in this illustration is  

$$
\mathbf{M}=
\left[ 
\begin{array}{ccc} 
m_{1,1} &  m_{1,2} & m_{1,3} & m_{1,4} \\ 
m_{2,1} &  m_{2,2} & m_{2,3} & m_{2,4} \\ 
m_{3,1} &  m_{3,2} & m_{3,3} & m_{3,4} \\ 
m_{4,1} &  m_{4,2} & m_{4,3} & m_{4,4} \\ 
\end{array} 
\right]
=
\left[ 
\begin{array}{ccc} 
1 &  4 & 0 & 2 \\ 
9 & 12 & 1 & 5 \\ 
20 & 3 & 4 & 2 \\ 
4 & 3 & 11 & 4 \\ 
\end{array} 
\right]
$$

### Question: how to recover matrix M from partially known elements and the gradients
Assuming that we only know the values of some matrix elements, while elements $$m_{1,2}$$, $$m_{2,2}$$, $$m_{3,2}$$, $$m_{1,3}$$, $$m_{2,3}$$, $$m_{3,3}$$ are unknown:  

$$
\left[ 
\begin{array}{ccc} 
1 &  ? & ? & 2 \\ 
9 & ? & ? & 5 \\ 
20 & ? & ? & 2 \\ 
4 & 3 & 11 & 4 \\ 
\end{array} 
\right]
$$

Also assuming that we have the knowledge of element-wise difference (which is a simple kind of gradient measures) along both vertical and horizonal directions:  

$$
\mathbf{M}_{d_{v}}=
\left[ 
\begin{array}{ccc} 
8 & 8 & 1 &  3 \\ 
11 & -9 & 3  & -3 \\ 
-16 &  0 & 7  & 2 \\ 
\end{array} 
\right]
$$

$$
\mathbf{M}_{d_h}=
\left[ 
\begin{array}{ccc} 
3 &  -4 &  2 \\ 
3 & -11  & 4 \\ 
-17 & 1 & -2 \\ 
-1 &  8 & -7 \\ 
\end{array} 
\right]
=
\left[ 
\begin{array}{ccc} 
3 &  3 &  -17 & -1 \\ 
-4 & -11 & 1 & 8 \\ 
2 &  4 & -2 & -7 \\ 
\end{array} 
\right]^T
$$  

_**Our goal**_ is to recover matrix $$\mathbf{M}$$ from the two gradient maps and patially known matrix elements.

### Finding a solution through matrix inversion
We could use matrix inversion to find matrix element values that simultaneously satisfy the vertical-gradient equation [1], the horizonal-gradient equation [2], and the boundary conditions [3]:

$$
\mathbf{D_{v}} \: \mathbf{m} = 
\left[ 
\begin{array}{ccc} 
8 & 8 & 1 &  3 & 
11 & -9 & 3  & -3 &
-16 &  0 & 7  & 2
\end{array} 
\right]^T
\: \: \: \to[1]
$$

$$
\mathbf{D_{h}} \: \mathbf{m} = 
\left[ 
\begin{array}{ccc} 
3 &  3 &  -17 & -1 & 
-4 & -11 & 1 & 8 &
2 &  4 & -2 & -7 
\end{array} 
\right]^T
\: \: \: \to[2]
$$  

$$
\mathbf{B} \: \mathbf{m} = 
\left[ 
\begin{array}{ccc} 
1 & 9 & 20 & 4 & 0 & 0 & 0 & 3 & 0 & 0 & 0 &  11 & 2 & 5 & 2 & 4
\end{array} 
\right]^T
\: \: \: \to[3]
$$  

where  

$$
\mathbf{D_{v}} = 
\left[ 
\begin{array}{ccc} 
-1 & 1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & -1 & 1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & -1 & 1 & 0 & 0 & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & -1 & 1 & 0 & 0 \\ 
0 & -1 & 1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & -1 & 1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & -1 & 1 & 0 & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & -1 & 1 & 0 \\ 
0 & 0 & -1 & 1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & -1 & 1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & -1 & 1 & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & -1 & 1 \\
\end{array} 
\right] 
\: \: \: \to[4]
$$

$$
\mathbf{D_{h}} = 
\left[ 
\begin{array}{ccc} 
-1 & 0 & 0 & 0 & 1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\
0 & -1 & 0 & 0 & 0 & 1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\
0 & 0 & -1 & 0 & 0 & 0 & 1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & -1 & 0 & 0 & 0 & 1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\ 
0 & 0 & 0 & 0 & -1 & 0 & 0 & 0 & 1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & -1 & 0 & 0 & 0 & 1 & 0 & 0 & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & -1 & 0 & 0 & 0 & 1 & 0 & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & -1 & 0 & 0 & 0 & 1 & 0 & 0 & 0 & 0 \\ 
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & -1 & 0 & 0 & 0 & 1 & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & -1 & 0 & 0 & 0 & 1 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & -1 & 0 & 0 & 0 & 1 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & -1 & 0 & 0 & 0 & 1 \\ 
\end{array} 
\right]
\: \: \: \to[5]
$$

$$
\mathbf{B}=
\left[ 
\begin{array}{ccc} 
1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\
0 & 1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\
0 & 0 & 1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & 1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 1 & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 1 & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 1 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 1 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 1 \\
\end{array} 
\right]
\: \: \: \to[6]
$$

$$
\mathbf{m} = 
\left[ 
\begin{array}{ccc} 
m_{1,1} \\  m_{2,1} \\ m_{3,1} \\ m_{4,1} \\ m_{1,2} \\  m_{2,2} \\ m_{3,2} \\ m_{4,2} \\ m_{1,3}  \\ m_{2,3}  \\ m_{3,3}  \\ m_{4,3}  \\ m_{1,4}  \\m_{2,4}  \\ m_{3,4} \\ m_{4,4} \\ 
\end{array} 
\right]
\: \: \: \to[7]
$$  

Note that Equations [4] to [6] could be combined, as shown in Equation [8]:

$$
\mathbf{E} \: \mathbf{m} = \mathbf{v} \: \: \: \to [8]
$$  

where $$\mathbf{E}$$ is the vertical concatenation of $$\mathbf{D_{v}}$$, $$\mathbf{D_{h}}$$, and $$\mathbf{B}$$;  
and $$\mathbf{v}$$ is the vertical concatenation of the following three column vectors:
$$
\left[ 
\begin{array}{ccc} 
8 & 8 & 1 &  3 & 
11 & -9 & 3  & -3 &
-16 &  0 & 7  & 2
\end{array} 
\right]^T
$$
,  
$$
\left[ 
\begin{array}{ccc} 
3 &  3 &  -17 & -1 & 
-4 & -11 & 1 & 8 &
2 &  4 & -2 & -7
\end{array} 
\right]^T
$$
,  
$$
\left[ 
\begin{array}{ccc} 
1 & 9 & 20 & 4 & 0 & 0 & 0 & 3 & 0 & 0 & 0 & 11 & 2 & 5 & 2 & 4
\end{array} 
\right]^T
$$  


The matrix $$\mathbf{m}$$ could be recovered with Equation [9]:  

$$
\mathbf{m} = (\mathbf{E})^{-1} \: \mathbf{v} \: \: \: \to [9]
$$

#### Julia codes:
``` julia

M = [1 4 0 2; 9 12 1 5; 20 3 4 2; 4 3 11 4.];
m = M[:];
Mdv = diff(M,dims=1);
Mdh = diff(M,dims=2);
xdim,ydim = size(M);
selectMatrix = ones(xdim,ydim);
selectMatrix[1,2]=0;
selectMatrix[1,3]=0;
selectMatrix[2,2]=0;
selectMatrix[2,3]=0;
selectMatrix[3,2]=0;
selectMatrix[3,3]=0;
M_measured = M.*selectMatrix;

let
    global Dv = Float64[]
    for cntx = 1:xdim-1
        for cnty = 1:ydim
            tmp = zeros(xdim,ydim)
            tmp[cntx,cnty]=-1
            tmp[cntx+1,cnty]=1;
            Dv = vcat(Dv,tmp[:]);
        end
    end
end
Dv = reshape(Dv,xdim*ydim,(xdim-1)*ydim,);
Dv = transpose(Dv);

let
    global Dh = Float64[]
    for cnty = 1:ydim-1
        for cntx = 1:xdim
            tmp = zeros(xdim,ydim)
            tmp[cntx,cnty]=-1
            tmp[cntx,cnty+1]=1;
            Dh = vcat(Dh,tmp[:]);
        end
    end
end
Dh = reshape(Dh,xdim*ydim,xdim*(ydim-1))
Dh = transpose(Dh);

using LinearAlgebra
B = diagm(0=>selectMatrix[:]);


E = vcat(Dv,Dh,B);
v = vcat(transpose(Mdv)[:],Mdh[:],M_measured[:]);

M_recovered = reshape(E\v,xdim,ydim);


```

### Implementation for different gradient measures

Instead of simply computing the element-wise difference, we could use different approaches to measure gradients. For example, the gradient maps along the vertical and horizontal directions, $$\mathbf{M}_{g_{v}}$$ and $$\mathbf{M}_{g_{h}}$$, could be computed with Equations [10] and [11], respectively, where $$vec$$ means reshaping a 2D matrix into a column vector.

$$
\mathbf{M}_{g_{v}}=\mathbf{G_{v}} \: \mathbf{m}=
\left[ 
\begin{array}{ccc} 
8 & 8 & 1 &  3 \\ 
9.5 & -0.5 & 2  & 0 \\ 
-2.5 &  -4.5 & 5  & -0.5 \\ 
-16 & 0 & 7 & 2 \\
\end{array} 
\right]_{vec} \: \: \: \to[10]
$$

$$
\mathbf{M}_{g_h}= \mathbf{G_{h}} \: \mathbf{m}=
\left[ 
\begin{array}{ccc} 
3 &  -0.5 & -1 &  2 \\ 
3 & -4 & -3.5 & 4 \\ 
-17 & -8 & -0.5 & -2 \\ 
-1 & 3.5 &  0.5 & -7 \\ 
\end{array} 
\right]_{vec} \: \: \: \to[11]
$$  

where

$$
\mathbf{G_{v}} = 
\left[ 
\begin{array}{ccc} 
-1 & 1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & -1 & 1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & -1 & 1 & 0 & 0 & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & -1 & 1 & 0 & 0 \\ 
-0.5 & 0 & 0.5 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & -0.5 & 0 & 0.5 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & -0.5 & 0 & 0.5 & 0 & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & -0.5 & 0 & 0.5 & 0 \\
0 & -0.5 & 0 & 0.5 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & -0.5 & 0 & 0.5 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & -0.5 & 0 & 0.5 & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & -0.5 & 0 & 0.5 \\
0 & 0 & -1 & 1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & -1 & 1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & -1 & 1 & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & -1 & 1 \\
\end{array} 
\right]
\: \: \: \to[12]
$$

$$
\mathbf{G_{h}} = 
\left[ 
\begin{array}{ccc} 
-1 & 0 & 0 & 0 & 1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\
0 & -1 & 0 & 0 & 0 & 1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\
0 & 0 & -1 & 0 & 0 & 0 & 1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & -1 & 0 & 0 & 0 & 1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\ 
-0.5 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0.5 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\
0 & -0.5 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0.5 & 0 & 0 & 0 & 0 & 0 & 0 \\
0 & 0 & -0.5 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0.5 & 0 & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & -0.5 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0.5 & 0 & 0 & 0 & 0 \\ 
0 & 0 & 0 & 0 & -0.5 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0.5 & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & -0.5 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0.5 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & -0.5 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0.5 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & -0.5 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0.5 \\ 
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & -1 & 0 & 0 & 0 & 1 & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & -1 & 0 & 0 & 0 & 1 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & -1 & 0 & 0 & 0 & 1 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & -1 & 0 & 0 & 0 & 1 \\ 
\end{array} 
\right]
\: \: \: \to[13]
$$

#### Julia codes:  
```julia
M = [1 4 0 2; 9 12 1 5; 20 3 4 2; 4 3 11 4.];
m = M[:];
Mgv = vcat(diff(M,dims=1)[1,:]',(diff(M[1:end-1,:],dims=1)+diff(M[2:end,:],dims=1))/2.,diff(M,dims=1)[end,:]');
Mgh = hcat(diff(M,dims=2)[:,1],(diff(M[:,1:end-1],dims=2)+diff(M[:,2:end],dims=2))/2.,diff(M,dims=2)[:,end]);
xdim,ydim = size(M);

selectMatrix = ones(xdim,ydim);
selectMatrix[1,2]=0;
selectMatrix[1,3]=0;
selectMatrix[2,2]=0;
selectMatrix[2,3]=0;
selectMatrix[3,2]=0;
selectMatrix[3,3]=0;
M_measured = M.*selectMatrix;

let
    global Gv = Float64[]
    for cntx = 1:xdim
        for cnty = 1:ydim
            tmp = zeros(xdim,ydim)
            if cntx == 1
                tmp[cntx,cnty]=-1
                tmp[cntx+1,cnty]=1;
            elseif cntx == xdim
                tmp[cntx-1,cnty]=-1
                tmp[cntx,cnty]=1;
            else
                tmp[cntx-1,cnty]=-0.5
                tmp[cntx+1,cnty]=0.5
            end
            Gv = vcat(Gv,tmp[:]);
        end
    end
end
Gv = reshape(Gv,xdim*ydim,xdim*ydim);
Gv = transpose(Gv);

let
    global Gh = Float64[]
    for cnty = 1:ydim
        for cntx = 1:xdim
            tmp = zeros(xdim,ydim)
            if cnty == 1
                tmp[cntx,cnty]=-1
                tmp[cntx,cnty+1]=1;
            elseif cnty == ydim
                tmp[cntx,cnty-1]=-1
                tmp[cntx,cnty]=1;
            else
                tmp[cntx,cnty-1]=-0.5
                tmp[cntx,cnty+1]=0.5;
            end
            Gh = vcat(Gh,tmp[:]);
        end
    end
end
Gh = reshape(Gh,xdim*ydim,xdim*ydim)
Gh = transpose(Gh);

using LinearAlgebra
B = diagm(0=>selectMatrix[:]);


E = vcat(Gv,Gh,B);
v = vcat(transpose(Mgv)[:],Mgh[:],M_measured[:]);

M_recovered = reshape(E\v,xdim,ydim);
```
