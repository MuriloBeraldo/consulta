  ---
  layout:artigo
  nome: triangulo_2.cpp - CG 2014 2S
  ---
  {%highlight cpp%}
  #include <windows.h>
    #include <windowsx.h>
    #include <d3d9.h>
    #include <d3dx9.h>

    #define SCREEN_WIDTH 800
    #define SCREEN_HEIGHT 600

    #pragma comment (lib, "d3d9.lib")
    #pragma comment (lib, "d3dx9.lib")

    LPDIRECT3D9 d3d;    
    LPDIRECT3DDEVICE9 d3ddev;    
    LPDIRECT3DVERTEXBUFFER9 v_buffer = NULL;    

    void initD3D(HWND hWnd);    
    void render_frame(void);    
    void cleanD3D(void);    
    void init_graphics(void);    

    struct CUSTOMVERTEX {FLOAT X, Y, Z; DWORD COLOR;};
    #define CUSTOMFVF (D3DFVF_XYZ | D3DFVF_DIFFUSE)

    LRESULT CALLBACK WindowProc(HWND hWnd, UINT message, WPARAM wParam, LPARAM lParam);


    int WINAPI WinMain(HINSTANCE hInstance,
                       HINSTANCE hPrevInstance,
                       LPSTR lpCmdLine,
                       int nCmdShow)
    {
        HWND hWnd;
        WNDCLASSEX wc;

        ZeroMemory(&wc, sizeof(WNDCLASSEX));

        wc.cbSize = sizeof(WNDCLASSEX);
        wc.style = CS_HREDRAW | CS_VREDRAW;
        wc.lpfnWndProc = WindowProc;
        wc.hInstance = hInstance;
        wc.hCursor = LoadCursor(NULL, IDC_ARROW);
        wc.lpszClassName = L"WindowClass";

        RegisterClassEx(&wc);

        hWnd = CreateWindowEx(NULL, L"WindowClass", L"Our Direct3D Program",
                              WS_OVERLAPPEDWINDOW, 0, 0, SCREEN_WIDTH, SCREEN_HEIGHT,
                              NULL, NULL, hInstance, NULL);

        ShowWindow(hWnd, nCmdShow);

        initD3D(hWnd);

        //Loop principal

        MSG msg;

        while(TRUE)
        {
            while(PeekMessage(&msg, NULL, 0, 0, PM_REMOVE))
            {
                TranslateMessage(&msg);
                DispatchMessage(&msg);
            }

            if(msg.message == WM_QUIT)
                break;

            render_frame();
        }

        cleanD3D();

        return msg.wParam;
    }


    LRESULT CALLBACK WindowProc(HWND hWnd, UINT message, WPARAM wParam, LPARAM lParam)
    {
        switch(message)
        {
            case WM_DESTROY:
                {
                    PostQuitMessage(0);
                    return 0;
                } break;
        }

        return DefWindowProc (hWnd, message, wParam, lParam);
    }

    void initD3D(HWND hWnd)
    {
        d3d = Direct3DCreate9(D3D_SDK_VERSION);

        D3DPRESENT_PARAMETERS d3dpp;

        ZeroMemory(&d3dpp, sizeof(d3dpp));
        d3dpp.Windowed = TRUE;
        d3dpp.SwapEffect = D3DSWAPEFFECT_DISCARD;
        d3dpp.hDeviceWindow = hWnd;
        d3dpp.BackBufferFormat = D3DFMT_X8R8G8B8;
        d3dpp.BackBufferWidth = SCREEN_WIDTH;
        d3dpp.BackBufferHeight = SCREEN_HEIGHT;

        d3d->CreateDevice(D3DADAPTER_DEFAULT,
                          D3DDEVTYPE_HAL,
                          hWnd,
                          D3DCREATE_SOFTWARE_VERTEXPROCESSING,
                          &d3dpp,
                          &d3ddev);

        init_graphics();

        d3ddev->SetRenderState(D3DRS_LIGHTING, FALSE);
    }


    void render_frame(void)
    {
        //d3ddev->Clear(0, NULL, D3DCLEAR_TARGET, D3DCOLOR_XRGB(0, 0, 0), 1.0f, 0);

        d3ddev->BeginScene();

        d3ddev->SetFVF(CUSTOMFVF);

       
        static float index = 0.0f; index-=20.55f;
        static float index2 = 0.0f; index2+=0.01f;

        D3DXMATRIX matRotateY;

        D3DXMatrixRotationZ(&matRotateY, index);



        D3DXMATRIX matScaleY;

        D3DXMatrixScaling(&matScaleY, 0.2f, 0.2f, 0.5f);

        D3DXMATRIX matTransY;

        D3DXMatrixTranslation(&matTransY, index2 , 0, 0);



        d3ddev->SetTransform(D3DTS_WORLD, &(matRotateY * matTransY * matRotateY * matRotateY));

        D3DXMATRIX matView;

        D3DXMatrixLookAtLH(&matView,
                           &D3DXVECTOR3 (0.0f, 0.0f, 10.0f),//pos
                           &D3DXVECTOR3 (0.0f, 0.0f, 0.0f),//ponto para o qual a camera esta olhando
                           &D3DXVECTOR3 (0.0f, 1.0f, 0.0f));//inclinacao da camera

        d3ddev->SetTransform(D3DTS_VIEW, &matView);

        D3DXMATRIX matProjection;

        D3DXMatrixPerspectiveFovLH(&matProjection,
                                   D3DXToRadian(45),    //angulo da lente
                                   (FLOAT)SCREEN_WIDTH / (FLOAT)SCREEN_HEIGHT, //aspect rate (4:3 , etc)
                                   1.0f,    //plano de corte proximo (near clipping pane)
                                   100.0f);    //plano de corte distante (far clipping pane)

        d3ddev->SetTransform(D3DTS_PROJECTION, &matProjection);    

        d3ddev->SetStreamSource(0, v_buffer, 0, sizeof(CUSTOMVERTEX));

        d3ddev->DrawPrimitive(D3DPT_TRIANGLESTRIP, 0, 2);

        d3ddev->EndScene();

        d3ddev->Present(NULL, NULL, NULL, NULL);
    }


    void cleanD3D(void)
    {
        v_buffer->Release();    
        d3ddev->Release();    
        d3d->Release();    
    }

    void init_graphics(void)
    {
        CUSTOMVERTEX vertices[] = 
        {
            { 3.0f, 3.0f, 0.0f, D3DCOLOR_XRGB(0, 255, 0), },
            { -3.0f, 3.0f, 0.0f, D3DCOLOR_XRGB(255, 0, 0), },
            { 3.0f, -3.0f, 0.0f, D3DCOLOR_XRGB(0, 0, 255), },
            { -3.0f, -3.0f, 0.0f, D3DCOLOR_XRGB(255, 0, 0), },
            
        };

        d3ddev->CreateVertexBuffer(4*sizeof(CUSTOMVERTEX),
                                   0,
                                   CUSTOMFVF,
                                   D3DPOOL_MANAGED,
                                   &v_buffer,
                                   NULL);

        VOID* pVoid;

        v_buffer->Lock(0, 0, (void**)&pVoid, 0);
        memcpy(pVoid, vertices, sizeof(vertices));
        v_buffer->Unlock();
    }
{%endhighlight%}