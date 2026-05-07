# srsUE
`05/06/2026 chunchioai1 130.127.199.183 as ue`  
srsRAN Project does not include aUE application. Hoever, srsRAN_4G does include a prototype 5G UE (srsUE).  
Executable file  
location: `srsRAN_4G/build/srsue/srsue`

Configuration file  
location: `srsRAN_Project/configs/gnb_rf_b210_fdd_srsUE.yml`  


## Modifications
The following changes need to be made to the UE configuration file to allow it to connect to the gNB in SA mode.
### Configure for B210
> [rf]  
> freq_offset = 0  
> tx_gain = 50  
> rx_gain = 40  
> srate = 23.04e6  
> nof_antennas = 1  
>   
> device_name = uhd  
> device_args = clock=external              # Use external reference clock with USRP B210.  
> time_adv_nsamples = 300  

### Disable the LTE and force the UE to use a 5G NR carrier
> [rat.eutra]  
> dl_earfcn = 2850  
> nof_carriers = 0  

### Configure for 5G SA mode operation
> [rat.nr]  
> bands = 3  
> nof_carriers = 1  
> max_nof_prb = 106                         # The setting have to be adapted for the used bandwidth according to table*  
> nof_prb = 106                             # The setting have to be adapted for the used bandwidth according to table*  

`BW/PRBs: 5/25, 10/52, 15/79, 20/106`  

### 
> [rrc]  
> release = 15  
> ue_catefory = 4  

### Default USIM Credentials
> [usim]  
> mode = soft  
> algo = milenage  
> opc  = 63BFA50EE6523365FF14C1F45F88737D  
> k    = 00112233445566778899aabbccddeeff  
> imsi = 001010123456780  
> imei = 353490069873319  

###
> [nas]  
> apn = srsapn  
> apn_protocol = ipv4  

# srsgNB
`05/06/2026 chunchioai2 130.127.199.61 as gNB`  
Executable file  
location: `srsRAN_Project/build/apps/gnb/gnb`  

Configuration file  
location: ``

## Modifications
The following changes need to be made to the **gNB** configuration file
### AMF
> cu_cp:  
>   amf:  
>     addr: 10.53.1.2                       # The address or hostname of the AMF. Check Open5GS config -> amf -> ngap -> addr  
>     port: 38412  
>     bind_addr: 10.53.1.1                  # A local IP that the gNB binds to for traffic from the AMF  
>     supported_tracking_aras:  
>       - tac: 7  
>         plmn_list:  
>           - plmn: "00101"  
>             tai_slice_support_list:  
>               - sst: 1  
>    inactivity_timer: 7200                 # Sets the UE/PDU Session/DRB inactivity timer to 7200 seconds, Supported: [1-7200].  

### Configure the RF front-end device
> ru_sdr:  
>   device_driver: uhd                      # The RF driver name.  
>   device_args: type=b200                  # Optinally pass arguments to the selected RF driver.  
>   clock: external                         # Use external reference clock with USTP B210.  
>   srate: 23.04                            # RF sample rate might need to be adjusted according to selected bandwidth.  
>   tx_gain: 75                             # Transmit gain of the RF might need to be adjusted to the given situation.  
>   rx_gain: 75

### Configure 5G cell parameters
> cell_cfg:  
>   dl_arfcn: 368500                        # ARFCN of the downlink carrier (center frequency).  
>   band: 3                                 # The NR band  
>   channel_bandwidth_MHz: 20               # Bandwidth in MHz. Number of PRBs will be automatically derived.  
>   common_scs: 15                          # Subcarrier spacing in kHz used for data.  
>   plmn: "00101"                           # PLMN broadcasted by the gNB.  
>   tac: 7                                  # Tracking area code (needs to match the core configuration).  
>   pdcch:  
>     common:  
>       ss0_index: 0                        # Set search space zero index ot match srs UE capabilities  
>       coreset0_index: 12                  # Set search CORESET Zero index to match srsUE capabilities  
>     dedicated:  
>       ss2_type: common                    # Search Space type, has to be set to common  
>       dci_format__1_and_1_1: false        # Set correct DCI format (fallback)  
>   prach:  
>     prach_config_index: 1                 # Sets PRACH config to match what is expected by srsUE  

# Running the Network
1. 5GC  
2. gNB  
3. UE  
