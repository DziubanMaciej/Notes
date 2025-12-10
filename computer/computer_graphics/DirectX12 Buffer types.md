parentTopic:: [[Computer graphics]]
links:: https://docs.microsoft.com/en-us/windows/win32/direct3dhlsl/d3d11-graphics-reference-sm5-objects

# DX12 Buffer types

Buffer type            | perfInConvergentAccess | perfInDivergentAccess | alignment | maxSize | maxElementSize         | readAlignment | readSize           | implicitConversion | swizzling | descriptor     | syntax
-----------------------|------------------------|-----------------------|-----------|---------|------------------------|---------------|--------------------|--------------------|-----------|----------------|----------------------------------------------
constantBuffer         | 1                      | 0                     | 256B      |  64KiB  | 64KiB (it's 1 element) | 1             | any                | 0                  | 0         | CBV            | `ConstantBuffer<SomeStruct>`
typedBuffer            | ?                      | ?                     | ?         |  ?      | Not limited            | 1             | dependsOnTypeArg   | 1                  | 0         | SRV            | `Buffer<uint4> buffer;` <br/> `buffer.Load(0);`
byteAddressBuffer      | ?                      | ?                     | ?         |  ?      | 4                      | 4             | 4,8,12,16          | 0                  | 0         | SRV (RAW flag) | `ByteAddressBuffer buffer` <br/> `buffer.Load3(16)`
structuredBuffer       | ?                      | ?                     | ?         |  ?      | 2048                   | ?             | dependsOnTypeArg   | 0                  | 1         | SRV            | `StructuredBuffer<SomeStruct>`
RwBuffer               |                        |                       |           |         |                        |               |                    |                    |           |                |
AppendBuffer           |                        |                       |           |         |                        |               |                    |                    |           |                |