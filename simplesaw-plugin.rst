LV2 Audio-Plugin Workshop OSCAMC
################################

:author: Christopher Arndt
:date: 2018-08-02


Simple Saw Plugin
-----------------

1. Initialize project with cookiecutter.

2. Copy the following files from
   https://github.com/SpotlightKid/MonoSynth/tree/master/plugins/monosynth into the
   ``plugins/SimpleSaw`` directory:

   * ``WaveTableOsc.{c,h}pp``
   * ``WaveUtils.{c,h}pp``

3. Add these objects to the ``OBJ_DSP`` list in the Makefile in ``plugins/SimpleSaw`` *before*
   ``PluginSimpleSaw.cpp.o``::

        WaveTableOsc.cpp.o \
        WaveUtils.cpp.o \

4. Add ``#include "WaveUtils.hpp"`` to ``plugins/SimpleSaw/PluginSimpleSaw.hpp``.

5. Add ``WaveTableOsc *osc;`` to the ``private`` Section of the ``PluginSimpleSaw`` class in
   ``plugins/SimpleSaw/PluginSimpleSaw.hpp``.

6. Add ``osc = sawOsc();`` in the constructor method of the ``PluginSimpleSaw`` class in
   ``plugins/SimpleSaw/PluginSimpleSaw.cpp``*before* the ``loadProgram(0);`` line.

7. Change every occurence of ``paramVolume`` to ``paramFrequency`` in
   ``plugins/SimpleSaw/PluginSimpleSaw.hpp`` and ``plugins/SimpleSaw/PluginSimpleSaw.cpp``.

8. Change the ``initParameter`` method of the ``PluginSimpleSaw`` class to set the name and symbol
   of the parameter to "Frequency" resp. "frequency" and its min/max/default values to 20.0/10000.0
   and 440.0;

9. Change the for loop in the ``run`` method of the ``PluginSimpleSaw`` class to the following::

        // output the oscillator waveform
        for (uint32_t i=0; i < frames; ++i) {
            outL[i] = osc->getOutput() * 0.8;
            outR[i] = osc->getOutput() * 0.8;
            osc->updatePhase();
        }

   and remove the lines initializing  the ``inpL``, ``inpR`` and ``vol`` variables.

10. Change the body of the ``setParametervalue`` method of the ``PluginSimpleSaw`` class to the
    following::

        fParams[index] = value;

        switch (index) {
            case paramFrequency:
                osc->setFrequency(value / fSampleRate);
                fParams[paramFrequency] = value;
                break;
        }

11. Change the body of the ``loadProgram`` method of the ``PluginSimpleSaw`` class to the
    following::

        switch (index) {
            case 0:
                setParameterValue(paramFrequency, 440.0f);
                break;
        }

    (Only the second parameter in the call to ``setParameterValue`` changes.)
