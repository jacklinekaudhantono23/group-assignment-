start_toolkit

d = epanet('water_map_analysis.inp');
d.plot;
d.plot('nodes', 'yes', 'fontsize', 12)
d.plot('links', 'yes', 'fontsize', 12)
d.plot('linksindex', 'yes', 'fontsize',8);
d.plot('nodesindex', 'yes', 'fontsize',14);
d.plot('links', 'yes', 'highlightlink',{'10'}, 'fontsize',8);        
d.plot('highlightnode',{'10', '32'});
d.plot('point', 'no');
d.plot('line', 'no');
linkSet1 = d.getLinkNameID(1:4);
colorLinkSet1 = repmat({'r'},1,length(linkSet1));
linkSet2 = d.getLinkNameID([5,6,7,8]);
colorLinkSet2=repmat({'g'},1,length(linkSet2));
d.plot('highlightlink',[linkSet1 linkSet2],'colorlink',[colorLinkSet1 colorLinkSet2])
nodeSet1 = d.getNodeNameID(1:4);
colorLinkSet1 = repmat({'y'},1,length(nodeSet1));
nodeSet2 = d.getNodeNameID([5,6,7,8]);
colorLinkSet2=repmat({'g'},1,length(nodeSet2));
d.plot('highlightnode',[nodeSet1 nodeSet2],'colornode',[colorLinkSet1 colorLinkSet2])
d.plot('legend', 'hide')
d.plot('extend', 'yes')
d.plot('legendposition', 'EastOutside')
h = figure;
d.plot('axes', h)
h = d.plotGraph
title('Network Graph');
d.plotGraph('nodes', 'yes');
title('Network Graph with node IDs');
d.unload
%% example 2
clear; close('all'); clc;
start_toolkit;
d = epanet('water_map_analysis.inp');
hrs = 100;
d.setTimeSimulationDuration(hrs*3600);
tic;
hyd_res_1 = d.getComputedTimeSeries
time_1 = toc;
tic;
hyd_res_2 = d.getComputedTimeSeries_ENepanet
time_2 = toc;
tic;
hyd_res_3 = d.getComputedHydraulicTimeSeries('time', 'flow', 'pressure'); 
time_3 = toc;
tic;
d.openHydraulicAnalysis;
d.initializeHydraulicAnalysis;
tstep=1;P=[];T_H=[];
while (tstep>0)
    t=d.runHydraulicAnalysis;
    P=[P; d.getNodePressure];
    T_H=[T_H; t];
    tstep=d.nextHydraulicAnalysisStep;
end
d.closeHydraulicAnalysis;
time_4 = toc;
d.unload

disp(['Elapsed time for the function `getComputedTimeSeries` is: ' num2str(time_1)])
disp(['Elapsed time for the function `getComputedTimeSeries_ENepanet` is: ' num2str(time_2)])
disp(['Elapsed time for the function `getComputedHydraulicTimeSeries` is: ' num2str(time_3)])
disp(['Elapsed time for the function `step-by-step` is: ' num2str(time_4)])
%% 3
clear; close('all'); clc;
start_toolkit;
d = epanet('water_map_analysis.inp');
qual_res = d.getComputedQualityTimeSeries
d.solveCompleteHydraulics;
d.openQualityAnalysis;
d.initializeQualityAnalysis;
tleft=1; P=[];T=[];QsN=[]; QsL=[]; 
while (tleft>0)
    t = d.runQualityAnalysis;
    P = [P; d.getNodePressure];
    QsN = [QsN; d.getNodeActualQuality];
    QsL = [QsL; d.getLinkQuality];
    T = [T; t];
    tleft = d.stepQualityAnalysisTimeLeft;
end
d.closeQualityAnalysis;

%% 4
clear; close('all'); clc;
start_toolkit;
d = epanet('water_map_analysis.inp');
hyd_res = d.getComputedHydraulicTimeSeries('time', 'pressure', 'flow', 'velocity');
hrs_time = hyd_res.Time/3600;
node_indices = [1, 3, 5];
node_names = d.getNodeNameID(node_indices);
units = d.getUnits;
for i=node_indices
    figure;
    plot(hrs_time, hyd_res .Pressure(:,i));
    title(['Pressure for the node id "', d.getNodeNameID{i},'"']);
    xlabel('Time (hrs)'); 
    ylabel(['Pressure (', units.NodePressureUnits,')'])
end
link_indices = [4, 8, 10];
link_names = d.getNodeNameID(link_indices);
for i=link_indices
    figure;
    plot(hrs_time, hyd_res.Velocity(:,i));
    title(['Velocity for the link id "', d.getLinkNameID{i},'"']);
    xlabel('Time (hrs)'); 
    ylabel(['Velocity (', units.LinkVelocityUnits,')'])
end
link_indices = [2, 3, 9];
for i=link_indices
    figure;
    plot(hrs_time, hyd_res.Flow(:,i));
    title(['Flow for the link id "', d.getLinkNameID{i},'"']);
    xlabel('Time (hrs)'); 
    ylabel(['Flow (', units.LinkFlowUnits,')'])
end
d.unload
%% 5
clear; close('all'); clc;
start_toolkit;
d = epanet('water_map_analysis.inp');
hyd_res = d.getComputedTimeSeries
times = hyd_res.Time;  
color = 'g';  
nodenameid = d.getNodeNameID;  
nodesconnlinks = d.getNodesConnectingLinksID; 
nodecoords = d.getNodeCoordinates; 
x = cell2mat(nodecoords(1));
y = cell2mat(nodecoords(2));
min_coorx = min(x);
min_coory = min(y);
max_coorx = max(x);
max_coory = max(y);
xsenoff = (max_coorx - min_coorx) * 0.075;
ysenoff = (max_coory - min_coory) * 0.03;
fig = d.plot('legend', 'hide');
format long g
time_step =1;
flow = hyd_res.Flow(time_step,:);
fontweight = 'bold';
fontsize = 10;
for i = d.getLinkIndex
    value = flow(i);    
    FromNode= find(strcmp(nodesconnlinks {i, 1}, nodenameid), 1);
    ToNode= find(strcmp(nodesconnlinks {i, 2}, nodenameid), 1);

    if~isempty(FromNode)
        x1 = double(nodecoords{1}(FromNode));
        y1 = double(nodecoords{2}(FromNode));
    end
    if ~isempty(ToNode)
        x2 = double(nodecoords{1}(ToNode));
        y2 = double(nodecoords{2}(ToNode));
    end
    F = line([x1 nodecoords{3}{i} x2], [y1 nodecoords{4}{i} y2], 'LineWidth', 1, 'Color', color, 'Parent', fig);
    text((x1+x2)/2-xsenoff, (y1+y2)/2+ysenoff, num2str(value, '%.5f'), 'FontWeight', fontweight, 'Fontsize', fontsize, 'color', color)
end

%% 
clear; close('all'); clc;
start_toolkit;

d1 = epanet('water_map_analysis.inp','ph'); 
d2 = epanet('hills.inp', 'ph');

disp('water_map_analysis - Elevations:')
disp('--------------------')
d1_Elevs = d1.getNodeElevations;
disp(d1_Elevs)
disp('hills - Elevations:')
disp('------------------')
d2_Elevs = d2.getNodeElevations;
disp(d2_Elevs)

disp('water_map_analysis - New Elevations:'); 
%% 
clear; close('all'); clc;
start_toolkit;

d = epanet('water_map_analysis.inp');

Results = d.getComputedTimeSeries;

nodeID = '10';
indexNode = d.getNodeIndex(nodeID);

figure;
subplot(2,1,1);
plot(Results.Time, Results.Pressure(:,indexNode));
title('Before set curve');
xlabel('Time (hrs)'); 
ylabel(['Pressure (', d.NodePressureUnits,')'])  

headCurve = d.getLinkPumpHeadCurveIndex;
curveIdx = d.getCurveIndex(headCurve);
d.setCurveValue(curveIdx,1,0,250);
d.setCurveValue(curveIdx,2,2000,0);

%d.setCurve(headCurve,[0 250; 2000 0]); 
%pumpIndex = d.getLinkPumpIndex
%d.getCurveIndex
%d.setLinkPumpHeadCurveIndex(pumpIndex, headCurve)

Results = d.getComputedTimeSeries;  
subplot(2,1,2);
plot(Results.Time, Results.Pressure(:,indexNode));
title('After set curve');
xlabel('Time (hrs)'); 
ylabel(['Pressure (', d.NodePressureUnits,')'])


d.unload



%% 
disp('----------------------')
d1.setNodeElevations(d1_Elevs + 200);
disp(d1.getNodeElevations)
disp('hills - New Elevations:')
disp('----------------------')
d2.setNodeElevations(d2_Elevs + 200);
disp(d2.getNodeElevations)

disp('water_map_analysis - Computed Time Series Values:')
disp('-----------------------------------')
disp(d1.getComputedTimeSeries)
disp('hills - Computed Time Series Values:')
disp('-----------------------------------')
disp(d2.getComputedTimeSeries)

d1.plot;
d2.plot;

d1.unload;
d2.unload;
%% 
clear; close('all'); clc;
start_toolkit;

d = epanet('Net1.inp');tic;

hours = 100;
d.setTimeSimulationDuration(hours*3600);

Results = d.getComputedTimeSeries
tocResults = toc;tic;

Hydraulics = d.getComputedHydraulicTimeSeries('flow', 'time')  
tocHydraulics = toc;tic;

Quality = d.getComputedQualityTimeSeries
tocQuality = toc;tic;

pipeindex=4;
nodeindex=6;

d.openHydraulicAnalysis;
d.initializeHydraulicAnalysis;
tstep=1;P=[];T_H=[];D=[];H=[];F=[];
while (tstep>0)
    t=d.runHydraulicAnalysis;
    P=[P; d.getNodePressure];
    D=[D; d.getNodeActualDemand];
    H=[H; d.getNodeHydraulicHead];
    F=[F; d.getLinkFlows];
    T_H=[T_H; t];
    tstep=d.nextHydraulicAnalysisStep;
    end
d.closeHydraulicAnalysis

d.openQualityAnalysis
d.initializeQualityAnalysis
tleft=1; P=[];T_Q=[];Q=[];  
sim_duration = d.getTimeSimulationDuration();
while (tleft>0 || t < sim_duration)
    t=d.runQualityAnalysis;
    Q=[Q; d.getNodeActualQuality];
    T_Q=[T_Q; t];
    tleft = d.stepQualityAnalysisTimeLeft;
end
d.closeQualityAnalysis;

d.unload

fprintf('\nSimulation duration: d hours\n', hours);
disp(['Run Time of function d.getComputedTimeSeries: ', num2str(tocResults), '(sec)'])
disp(['Run Time of function d.getComputedHydraulicTimeSeries: ', num2str(tocHydraulics), '(sec)'])
disp(['Run Time of function d.getComputedQualityTimeSeries: ', num2str(tocQuality), '(sec)'])

figure;
plot(Results.Time, Results.Flow(:,pipeindex));
title('d.getComputedTimeSeries (Ignore events)');
xlabel('Time (sec)'); ylabel(['Flow (',d.LinkFlowUnits,') - Link ID "',d.LinkNameID{pipeindex},'"'])
set(gca,'XTickLabel',num2str(get(gca,'XTick').'));

figure;
plot(Hydraulics.Time, Hydraulics.Flow(:,pipeindex));
title('d.getComputedHydraulicTimeSeries');
xlabel('Time (sec)'); ylabel(['Flow (',d.LinkFlowUnits,') - Link ID "',d.LinkNameID{pipeindex},'"'])
set(gca,'XTickLabel',num2str(get(gca,'XTick').'));

figure;
plot(T_H, F(:,pipeindex));
title('step by step Hydraulic Analysis');
xlabel('Time (sec)'); ylabel(['Flow (',d.LinkFlowUnits,') - Link ID "',d.LinkNameID{pipeindex},'"'])
set(gca,'XTickLabel',num2str(get(gca,'XTick').'));

figure;
plot(Results.Time, Results.NodeQuality(:,nodeindex));
title('d.getComputedTimeSeries (Ignore events)');
xlabel('Time (sec)'); ylabel(['Node Quality (',d.QualityChemUnits,') - Node ID "',d.NodeNameID{nodeindex},'"'])
set(gca,'XTickLabel',num2str(get(gca,'XTick').'));z



 
