# Chapter 2

```C
\\ simple c operation
void vecAdd(float *h_A, float *h_B, float *h_C, int n) {
    for (int i=0; i<n; i++) h_C[i] = h_A[i] + h_B[i];
}

int main() {
    vecAdd(h_A, h_B, h_C, N);
}
```

the user needs to transfer memory from the host device to the GPU for computation and back to
the host after the computation. Data transfer is usually done through 
1. `cudaMalloc`
2. `cudaFree`
3. `cudaMemcpy`

```C
float *d_A;
int size= n*sizeof(float);
cudaMalloc((void**)&d_A, size);
\\ cuda operation
cudaFree(d_A);
```

### usage of cuda memory copy

```C
cudaMemcpy(d_A, size, cudaMemcpyHostToDevice);
cudaMemcpy(d_B, size, cudaMemcpyHostToDevice);
C = d_A + d_B;
cudaMemcpy(C, size, cudaMemcpyDeviceToHost);
```

### Error Checking to catch memory allocation errors
```C
CudaError_t err = CudaMalloc((void **) &d_A, size);
if (error != CudaSuccess) {
    printf("%s in %s at line %d\n", cudaGetErrorString(err).__FILE__.__LINE__);
    exit(EXIT_FAILURE);
}
```

TODO: add verbiage on how threads are divided

## Vector addition kernel

```C
__global__ void vecAddKernel(float *A, float *B, float *C, int n) {
    int i = blockDim.x * blockIdx.x + threadIdx.x;
    if (i<n) C[i] = A[i] + B[i];
}

void vecAdd(float *A, float *B, float *C, int n) {
    int size = n * sizeof(float);
    float *d_A, *d_B, *d_C;

    cudaMalloc((void **) &d_A, size);
    cudaMemcpy(d_A, A, size, cudaMemcpyHostToDevice);
    cudaMalloc((void **) &d_B, size);
    cudaMemcpy(d_B, B, size, cudaMemcpyHostToDevice);

    cudaMalloc((void **) &d_C, size);

    vecAddKernel<<<ceil(n/256.0), 256>>>(d_A, d_B, d_C, n);
    cudaMemcpu(C, d_C, size, cudaMemcpyDeviceToHost);

    cudaFree(d_A); cudaFree(d_B); cudaFree(d_C);
}

```


