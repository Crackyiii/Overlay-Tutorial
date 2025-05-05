# Overlay-Tutorial

bool overlay_class::installImGui( )
{
	WNDCLASSEX wc = { sizeof(WNDCLASSEX) };
	wc.style = CS_HREDRAW | CS_VREDRAW;
	wc.lpfnWndProc = WndProc;
	wc.hInstance = GetModuleHandle(NULL);
	wc.lpszClassName = L"Fortnite";
	RegisterClassEx(&wc);

	window.WHandle = CreateWindowEx( WS_EX_TOPMOST | WS_EX_LAYERED | WS_EX_TRANSPARENT, L"Fortnite", L"Fortnite", WS_POPUP, 0, 0, window.ScreenWidth, window.ScreenHeight, NULL, NULL, GetModuleHandle(NULL), 
  NULL);

	if (!window.WHandle)
		return false;

	SetLayeredWindowAttributes(window.WHandle, RGB(0, 0, 0), 255, LWA_ALPHA);

	LONG ex_style = GetWindowLong(window.WHandle, GWL_EXSTYLE);
	ex_style |= WS_EX_TRANSPARENT | WS_EX_LAYERED;
	SetWindowLong(window.WHandle, GWL_EXSTYLE, ex_style);

	ex_style |= WS_EX_TRANSPARENT;
	SetWindowLong(window.WHandle, GWL_EXSTYLE, ex_style);

	MARGINS margins = { -1 };
	DwmExtendFrameIntoClientArea(window.WHandle, &margins);

	ShowWindow(window.WHandle, SW_SHOW);
	UpdateWindow(window.WHandle);

	DXGI_SWAP_CHAIN_DESC swap_chain_description;
	ZeroMemory(&swap_chain_description, sizeof(swap_chain_description));
	swap_chain_description.BufferCount = 2;
	swap_chain_description.BufferDesc.Width = window.ScreenWidth;
	swap_chain_description.BufferDesc.Height = window.ScreenHeight;
	swap_chain_description.BufferDesc.Format = DXGI_FORMAT_R8G8B8A8_UNORM;
	swap_chain_description.BufferDesc.RefreshRate.Numerator = 60;
	swap_chain_description.BufferDesc.RefreshRate.Denominator = 1;
	swap_chain_description.Flags = DXGI_SWAP_CHAIN_FLAG_ALLOW_MODE_SWITCH;
	swap_chain_description.BufferUsage = DXGI_USAGE_RENDER_TARGET_OUTPUT;
	swap_chain_description.OutputWindow = window.WHandle;
	swap_chain_description.SampleDesc.Count = 1;
	swap_chain_description.SampleDesc.Quality = 0;
	swap_chain_description.Windowed = TRUE;
	swap_chain_description.SwapEffect = DXGI_SWAP_EFFECT_DISCARD;

	D3D_FEATURE_LEVEL feature_level;
	D3D_FEATURE_LEVEL feature_level_array[2] = { D3D_FEATURE_LEVEL_11_0, D3D_FEATURE_LEVEL_10_0 };

	if (FAILED(D3D11CreateDeviceAndSwapChain(NULL, D3D_DRIVER_TYPE_HARDWARE, NULL, 0, feature_level_array, 2, D3D11_SDK_VERSION, &swap_chain_description, &D3D_Swap_Chain, &D3D_Device, &feature_level, &D3D_Device_ctx)))
	{
		return false;
	}

	ID3D11Texture2D* back_buffer;
	D3D_Swap_Chain->GetBuffer(0, IID_PPV_ARGS(&back_buffer));
	D3D_Device->CreateRenderTargetView(back_buffer, NULL, &D3D_Render_Target);
	back_buffer->Release();

	IMGUI_CHECKVERSION();
	ImGui::CreateContext();
	ImGuiIO& io = ImGui::GetIO();
	io.ConfigFlags |= ImGuiConfigFlags_NavEnableKeyboard;
	io.ConfigFlags |= ImGuiConfigFlags_NavEnableGamepad;

	ImGui_ImplWin32_Init(window.WHandle);
	ImGui_ImplDX11_Init(D3D_Device, D3D_Device_ctx);

	MenuFont = io.Fonts->AddFontFromFileTTF(("C:\\Windows\\Fonts\\impact.ttf"), 18.f);

	return true;
}

bool overlay_class::renderLoop( )
{
	MSG msg = { NULL };
	memset(&msg, 0, sizeof(MSG));

	while (msg.message != WM_QUIT)
	{
		UpdateWindow(window.WHandle);
		ShowWindow(window.WHandle, SW_SHOW);

		if (PeekMessageA(&msg, window.WHandle, 0, 0, PM_REMOVE))
		{
			TranslateMessage(&msg);
			DispatchMessageA(&msg);
		}

		ImGuiIO& io = ImGui::GetIO();
		io.DeltaTime = 1.0f / 60.0f;

		POINT p_cursor;
		GetCursorPos(&p_cursor);
		io.MousePos.x = p_cursor.x;
		io.MousePos.y = p_cursor.y;

		if (GetAsyncKeyState(VK_LBUTTON) & 0x8000)
		{
			io.MouseDown[0] = true;
			io.MouseClicked[0] = true;
			io.MouseClickedPos[0] = io.MousePos;
		}
		else
		{
			io.MouseDown[0] = false;
		}

		overlay.drawLoop();
	}

	ImGui_ImplDX11_Shutdown();
	ImGui_ImplWin32_Shutdown();
	ImGui::DestroyContext();

	DestroyWindow(window.WHandle);

	return true;
}
