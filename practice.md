import os                           
import scipy                        
import numpy as np                   
import matplotlib.pyplot as plt      
from tkinter import filedialog 

###Optional for old method of calcualting Z component
from scipy.signal import find_peaks
from scipy.interpolate import interp1d
from scipy.ndimage import uniform_filter1d as smooth

                    title='Open files',
                    initialdir='C:\\Users\\Brayd\\OneDrive\\1. 300 data\\300 matlab files\\')

    ###Store average components of magnetization if desired   
    x_mag = []
    y_mag = []
    z_mag = []
    
    ###Voltages for plotting things vs the Z/AC field strength
    voltages = np.linspace(100, 1000, 10)
    
    ###Iterate over the selected files with their indices.
    for idx, path in enumerate(paths):
        print(f"Displayed is number {idx} of the selected files. ")
        data_dict = scipy.io.loadmat(path)
        t = data_dict["time_axis"].flatten()  # time axis (seconds)
        amp = data_dict["pulseAmp"].flatten()  # pulse amplitude (a. u.)
        relPhase = data_dict["relPhase"].flatten()  # relative phase (rad)
        
        ###Normalize amplitude to value closest to 1.3 s which is usually shortly after Z/AC field turns on
        idx_1_3 = np.argmin(np.abs(t - 1.3))
        amp_ref = amp[idx_1_3]
        amp_norm = amp / amp_ref
        
        ###Specify a time interval by creating a Boolean array (mask)
        mask = (t >= 4.2) & (t<= 4.8)
        t_mask=t[mask]
        amp_mask=amp_norm[mask]
        phase_mask=relPhase[mask]
        
        ###Compuse X and Y components based on normalized amplitude
        Ix=amp_mask*np.cos(phase_mask)
        Iy=amp_mask*np.sin(phase_mask)
        
        ###Optional Z calcualtion based on method from first trajectories paper
        peak_times,_=find_peaks(amp_mask) #Find maxima
        spline=interp1d(t_mask[peak_times],amp_mask[peak_times],fill_value='extrapolate') #Curve extrapolated through maxima
        ref_amp=spline(t_mask)+0.05 #Shift the spline fit upward
        ref_amp = smooth(ref_amp,100) #Smooth the spline fit
        Iz = np.sqrt(ref_amp**2 - Ix**2 - Iy**2) #
        
        print(data_dict)


        ###Take average of Ix and Iy when range(...) specifys values of i e.g., range(2) give [0,1]
        xvals=[np.mean(Ix[i::2]) for i in range(2)]
        yvals=[np.mean(Iy[i::2]) for i in range(2)]
        
        ##Colors for when plotting four pt trajectories
        colors_list = ['navy', 'maroon', 'lightseagreen', 'grey']        
        colors = [colors_list[i % len(colors_list)] for i in range(len(t_mask))]
        

        
        ###Calculate magnitude of the change in Ix when Ix is oscillating
        x_magnitude = np.max(Ix) - np.min(Ix)
        x_mag.append(x_magnitude)
        
        ###Calculate magnitude of the change in Iy when Ix is oscillating
        y_magnitude = np.max(Iy) - np.min(Iy)
        y_mag.append(y_magnitude)
        
        ###Optional plot to look at phase, ampltitude etc
        # plt.figure(figsize=(2, 2))
        # plt.tick_params(axis='both', labelsize=30) 
        # plt.scatter(t_mask, phase_mask, s=30, c=colors)
        # plt.xlabel('Time (s)', fontsize=30)
        # plt.ylabel('Phase (rad)', fontsize=30)
        # # plt.xlim(-0.25,1)
        # # plt.ylim(-1,1)
        # # plt.title(f"Path {path}")
        # # plt.title('Phase vs Time', fontsize=30)
        # plt.show()
        
        
        # ###Plot the XY projection of the average of Ix and Iy over a given time interval
        # plt.figure(figsize=(1, 1))
        # plt.tick_params(axis='both', labelsize=60) 
        # plt.scatter(xvals, yvals, s=1000, c='black')
        # plt.xlabel('X', fontsize=60)
        # plt.ylabel('Y', fontsize=60)
        # plt.xlim(0.75,1)
        # plt.ylim(-1,1)
        # plt.title(f"Path {path}")
        # # # plt.title('Phase vs Time', fontsize=30)
        # plt.show()
        
        

        
        ###Define the 16 colors for trajectorues with 16 vertices
        # color1 = 'navy'
        # color2 = 'maroon'
        # color3 = 'grey'
        # color4 = 'olivedrab'
        # color5 = 'forestgreen'
        # color6 = 'slateblue'
        # color7 = 'teal'
        # color8 = 'crimson'
        # color9 = 'darkorange'
        # color10 = 'indigo'
        # color11 = 'darkviolet'
        # color12 = 'chocolate'
        # color13 = 'gold'
        # color14 = 'plum'
        # color15 = 'seashell'
        # color16 = 'lightseagreen'

        # colors_list = [
        #     color1, color2, color3, color4, color5, color6, color7, color8,
        #     color9, color10, color11, color12, color13, color14, color15, color16
        # ]

        # colors = [colors_list[i % 16] for i in range(len(t_mask))]

        
        ##For point trajectories
        colors_list = [
            'navy', 'maroon'
        ]

        colors = [colors_list[i % len(colors_list)] for i in range(len(t_mask))]



        

        ###Plot groups of 4 points in the same color
        # colors_list = ['navy', 'maroon'] 

        # # Assign colors in repeating groups of 4
        # colors = [colors_list[(i // 4) % len(colors_list)] for i in range(len(t_mask))]

        
        ###Create plot with subplots
        fig, axs = plt.subplots(2, 3, figsize=(15, 6))
        
        plt.rcParams.update({'xtick.labelsize': 20, 
                     'ytick.labelsize': 20})

        #Plot amplitude
        axs[0,0].scatter(t_mask, amp_mask, c=colors)
        axs[0,0].set_title('Amplitude vs Time',fontsize=28)
        axs[0,0].set_xlabel('Time (s)',fontsize=28)
        axs[0,0].set_ylabel('Amplitude',fontsize=28)
        
        #Plot Ix
        axs[0,1].scatter(t_mask, Ix, c=colors)
        axs[0,1].set_title('Ix vs Time',fontsize=28)
        axs[0,1].set_xlabel('Time (s)',fontsize=28)
        axs[0,1].set_ylabel('Ix',fontsize=28)
        
        #Plot Iy
        axs[0,2].scatter(t_mask, Iy, c=colors)
        axs[0,2].set_title('Iy vs Time', fontsize=28)
        axs[0,2].set_xlabel('Time',fontsize=28)
        axs[0,2].set_ylabel('Iy',fontsize=28)
                
        #Plot phase
        axs[1,0].scatter(t_mask, phase_mask, c=colors)
        axs[1,0].set_title('Phase vs Time',fontsize=28)
        axs[1,0].set_xlabel('Time (s)',fontsize=28)
        axs[1,0].set_ylabel('Phase (rad)',fontsize=28)
        
        #XY projection with color map for time
        scatter4 = axs[1, 1].scatter(Ix[0::2], Iy[0::2], c=t_mask[0::2], s=100, cmap='seismic')
        axs[1,1].set_title('XY',fontsize=28)
        axs[1,1].set_xlabel('X',fontsize=28)
        axs[1,1].set_ylabel('Y',fontsize=28)
        # axs[1,1].set_ylim(0.97, 0.75)
        fig.colorbar(scatter4, ax=axs[1, 1], label='time')
        
        # XY projection without color map
        axs[1,2].scatter(Ix, Iy, c=colors)
        axs[1,2].set_title('XY projection',fontsize=28)
        axs[1,2].set_xlabel('X',fontsize=28)
        axs[1,2].set_ylabel('Y',fontsize=28)

        plt.tight_layout()
        plt.show()
        
        
        ##Fourier Transform of amplitude
        fs = 1 / (t_mask[0::2][1] - t_mask[0::2][0])  # Sampling frequency (assumes uniform spacing)
        N = len(amp_mask[0::2])                 # Number of samples
        fft_amp = np.fft.fft(amp_mask[0::2])    # Compute FFT
        fft_freq = np.fft.fftfreq(N, d=1/fs)  # Frequency axis
        print(1/fs)

        #Only keep the positive frequencies (real part of spectrum)
        pos_mask = fft_freq >= 0
        fft_amp = np.abs(fft_amp[pos_mask])
        fft_freq = fft_freq[pos_mask]

        #Normalize the FFT amplitude
        fft_amp = fft_amp / np.max(fft_amp)

        plt.figure()
        plt.plot(fft_freq, fft_amp, linewidth=3, color='navy')
        # plt.xlim(100,400)
        # plt.ylim(0,0.1)
        plt.xlabel('Frequency (Hz)', fontsize=30)
        plt.ylabel('Amplitude (a.u.)', fontsize=30)
        plt.show()
  
    
    
    ###For plotting things such as Ix as a function of Z/AC amplitude
    # plt.figure(figsize=(10, 18))  # Wider for legends on the side
    
    # # X Magnetization
    # plt.subplot(3, 1, 1)
    # plt.tick_params(axis='both', labelsize=30) 
    # plt.plot(voltages, x_mag, '-o', markersize=20, linewidth=4, c='forestgreen')
    # plt.xlabel('Voltage (mV)', fontsize=30)
    # plt.ylabel('Ix change (a.u.)', fontsize=30)
    # plt.title('Change in Ix', fontsize=30)
    
    # plt.subplot(3, 1, 2)
    # plt.tick_params(axis='both', labelsize=30) 
    # plt.plot(voltages, y_mag, '-o', markersize=20, linewidth=4, c='navy')
    # plt.xlabel('Voltage (mV)', fontsize=30)
    # plt.ylabel('Iy change (a.u.)', fontsize=30)
    # plt.title('Change in Iy', fontsize=30)
    
    # plt.subplot(3, 1, 3)
    # plt.tick_params(axis='both', labelsize=30) 
    # plt.plot(voltages, z_mag, '-o', markersize=20, linewidth=4, c='maroon')
    # plt.xlabel('Voltage (mV)', fontsize=30)
    # plt.ylabel('Iz change (a.u.)', fontsize=30)
    # plt.title('Change in Iz', fontsize=30)
    
    # plt.tight_layout()
    # plt.show()
    
    plt.scatter(t_mask, phase_mask)
    plt.show()
    
    
    
     
# Check if the script is being run directly (not being imported as a module).
if __name__ == "__main__":
    # Call the main function to start the program.
    main()
