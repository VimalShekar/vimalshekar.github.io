---
layout: post
title: Enumerating Terminal Sessions and listing CPU usage
date: 2017-12-12
category: CodeSamples
---

Recently, I was interested in enumerating all the sessions that are active on a terminal server and figuring out which sessions used the least CPU. Enumerating TS sessions is relatively easy to do and there are a couple of ways to do it. 

1. You can use the WTSEnumerateSessionsEx API, to get back an array of WTS_SESSION_INFO_1 structures. The advantage of using WTS API is that you can open a handle to a remote machine and enumerate sessions on that machine.

    //-- Enumerate TS sessions
    WTSEnumerateSessionsEx( 
        WTS_CURRENT_SERVER_HANDLE ,  //<-- To enumerate sessions on the current server>
        &dwLevel, 
        1, 
        &pSessionInfo,  //<-- pointer to the array of WTS_SESSION_INFO_1 structures
        &dwCount);      //<-- number of elements in the array.


2. The second method is to use LsaEnumerateLogonSessions() API to get the logon session LUIDs and then use LsaGetLogonSessionData() API to get details of each session. We'll do this on another day.

Once you have enumerated the sessions and you can make a WMI query to "Win32_PerfFormattedData_TermService_TerminalServicesSession" class to retrieve performance data from ther Terminal Service provider. This may seem complicated, but once you get the hang of it - its easy. I have a code sample for this here: https://github.com/VimalShekar/Cpp/blob/master/src/termsessioncpu/terminalsessionlist.cpp


Here's a quick walkthrough:
1. The first step is to initialize the COM infrastructure.

    //-- Initialize COM and set Security 
    if (FAILED (hr = CoInitializeEx(NULL,COINIT_MULTITHREADED)))
    {
        printf("\n COM initialization failed with error: %d", hr);
        goto __cleanup;
    }

    //-- Set the COM security to default
    if (FAILED (hr = CoInitializeSecurity( NULL, -1, NULL, NULL, RPC_C_AUTHN_LEVEL_NONE, RPC_C_IMP_LEVEL_IMPERSONATE, NULL, EOAC_NONE, 0)))
    {
        printf("\n COM Security initialization failed with error: %d", hr);
        goto __cleanup;
    }

2. Next, create an instance of the Wbem locator and connect to the "root\cimv2" WMI namespace. If we were connecting to a remote machine, the namespace name would be "\\\\<server>\\root\\cimv2"

    if (FAILED (hr = CoCreateInstance( CLSID_WbemLocator, NULL, CLSCTX_INPROC_SERVER, IID_IWbemLocator, (void**) &pWbemLocator)))
    {
        printf("\n Failed to create instance of IWbemLocator. Error: %d", hr);
        goto __cleanup;
    }

    //-- Connect to Root\CIMv2 namespace
    if (FAILED (hr = pWbemLocator->ConnectServer( bstrNameSpace, NULL, NULL, NULL, 0L, NULL, NULL, &pNameSpace)))
    {
        printf("\nNot able to connect to the namespace: %d", hr);
        goto __cleanup;
    }

3. Create an instance of WbemRefresher class. This class helps us retrieve performance data that is periodically refreshed. Use QueryInterface method to retrieve its IWbemConfigureRefresher interface. We will use this interface to configure the WbemRefresher.


    //-- Create a WbemRefresher instance
    if (FAILED (hr = CoCreateInstance( CLSID_WbemRefresher, NULL, CLSCTX_INPROC_SERVER, IID_IWbemRefresher,  (void**) &pRefresher)))
    {
        printf("\n failed to create WbemRefresher object : %d", hr);
        goto __cleanup;
    }

    //-- Get the Config interface to configure the refresher
    if (FAILED (hr = pRefresher->QueryInterface(IID_IWbemConfigureRefresher, (void **)&pConfig)))
    {
        printf("\n failed to create IWbemConfigureRefresher object : %d", hr);
        goto __cleanup;
    }
    

4. Add an Enumerator object for the namespace and class that we are interested in. In this case  "Win32_PerfFormattedData_TermService_TerminalServicesSession" class. The AddEnum() method of the IWbemConfigureRefresher returns a pointer to the IWbemHiPerfEnum.

    pConfig->AddEnum( pNameSpace, L"Win32_PerfFormattedData_TermService_TerminalServicesSession", 0, NULL, &pEnum, &lID);


5. Now we can call the pRefresher->Refresh(0L) method to get refreshed data any number of times. Use the enumerator to Get the value of the objects. 


        //-- Refresh the data source
        pRefresher->Refresh(0L);

        //-- call once to get the size of buffers to be allocated, 
        //-- Make the necessary allocations and then call again to retried the data
        hr = pEnum->GetObjects(0L, dwNumObjects, apEnumAccess, &dwNumReturned);
        if (hr == WBEM_E_BUFFER_TOO_SMALL && dwNumReturned > dwNumObjects)
        {
            //-- Allocate a bigger buffer and retry call.
            apEnumAccess = new IWbemObjectAccess*[dwNumReturned];
            if (NULL == apEnumAccess)
            {
                hr = E_OUTOFMEMORY;
                goto __cleanup;
            }

            SecureZeroMemory(apEnumAccess, dwNumReturned*sizeof(IWbemObjectAccess*));
            dwNumObjects = dwNumReturned;

            if (FAILED (hr = pEnum->GetObjects(0L, dwNumObjects, apEnumAccess, &dwNumReturned)))
            {
                printf("\n Failed to get objects: %d", hr);
                goto __cleanup;
            }
        } 

6. GetObjects() returns an array of objects. Each object in the array has attributes, first you a handle to the attribute's location (this is sort of like getting an offset into where the attribute is within the object). 

    apEnumAccess[0]->GetPropertyHandle( L"PercentPrivilegedTime", &PercentPrivilegedTime_type, &lPercentPrivilegedTime_value);
    apEnumAccess[0]->GetPropertyHandle( L"PercentProcessorTime", &PercentProcessorTime_type, &lPercentProcessorTime_value);
    apEnumAccess[0]->GetPropertyHandle( L"PercentUserTime", &PercentUserTime_type, &lPercentUserTime_value);
    apEnumAccess[0]->GetPropertyHandle( L"Name", &SessionName, &lSessionName);

7. Now that you have handles to all the attributes you are interested in, you can iterate through the object to retrieve the attributes you are interested in and print them.

8. When it comes to cleaning up - again remember that  GetObjects() returns an array of COM objects. So you have to Release() each object in the array by iterating over them, otherwise you would be leaking memory.

        //-- cleanup apEnumAccess
        if (NULL != apEnumAccess)
        {
            for (DWORD i = 0; i < dwNumReturned; i++)
            {
                if (apEnumAccess[i] != NULL)
                {
                    apEnumAccess[i]->Release();
                    apEnumAccess[i] = NULL;
                }
            }
            delete [] apEnumAccess;
            apEnumAccess = NULL;
        }

9. You can loop through steps 5,6,7 and 8 any number of times, until you have all the samples you need to deterministically tell which sessions are idle. Finally clean up all the COM objects you have instantiated.

Refer to the full sample on  https://github.com/VimalShekar/Cpp/blob/master/src/termsessioncpu/terminalsessionlist.cpp
