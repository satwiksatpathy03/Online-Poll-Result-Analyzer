# Online-Poll-Result-Analyzer
The Online Poll Result Analyzer is a web-based application designed to create, manage, and analyze polls efficiently. It allows administrators to set up polls, users to vote online, and provides real-time result analysis through graphical and statistical reports

import React, { useState, useMemo } from 'react';
import { PieChart, Pie, Cell, BarChart, Bar, XAxis, YAxis, CartesianGrid, Tooltip, Legend, ResponsiveContainer } from 'recharts';
import { Download, Plus, Trash2, BarChart3, TrendingUp } from 'lucide-react';

const PollAnalyzer = () => {
  const [pollQuestion, setPollQuestion] = useState('Preferred Learning Mode');
  const [responses, setResponses] = useState([
    { option: 'Online', votes: 45 },
    { option: 'Offline', votes: 25 },
    { option: 'Hybrid', votes: 30 }
  ]);
  const [chartType, setChartType] = useState('pie');
  const [newOption, setNewOption] = useState('');
  const [newVotes, setNewVotes] = useState('');

  const analysisData = useMemo(() => {
    const totalVotes = responses.reduce((sum, item) => sum + item.votes, 0);
    const dataWithPercentages = responses.map(item => ({
      ...item,
      percentage: totalVotes > 0 ? ((item.votes / totalVotes) * 100).toFixed(1) : 0
    }));
    
    const winner = dataWithPercentages.reduce((max, item) => 
      item.votes > max.votes ? item : max, dataWithPercentages[0] || { option: 'None', votes: 0 }
    );

    return { dataWithPercentages, totalVotes, winner };
  }, [responses]);

  const colors = ['#8884d8', '#82ca9d', '#ffc658', '#ff7c7c', '#8dd1e1', '#d084d0'];

  const addResponse = () => {
    if (newOption.trim() && newVotes && !isNaN(Number(newVotes))) {
      setResponses([...responses, { 
        option: newOption.trim(), 
        votes: parseInt(newVotes) 
      }]);
      setNewOption('');
      setNewVotes('');
    }
  };

  const removeResponse = (index) => {
    setResponses(responses.filter((_, i) => i !== index));
  };

  const updateVotes = (index, votes) => {
    const updatedResponses = [...responses];
    updatedResponses[index].votes = Math.max(0, parseInt(votes) || 0);
    setResponses(updatedResponses);
  };

  const exportReport = () => {
    const reportData = {
      question: pollQuestion,
      totalVotes: analysisData.totalVotes,
      results: analysisData.dataWithPercentages,
      winner: analysisData.winner
    };
    
    const blob = new Blob([JSON.stringify(reportData, null, 2)], { type: 'application/json' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = 'poll-results.json';
    a.click();
    URL.revokeObjectURL(url);
  };

  return (
    <div className="max-w-6xl mx-auto p-6 bg-gray-50 min-h-screen">
      <div className="bg-white rounded-lg shadow-lg p-6 mb-6">
        <h1 className="text-3xl font-bold text-gray-800 mb-2">Online Poll Result Analyzer</h1>
        <p className="text-gray-600">Visualize and analyze poll results with interactive charts</p>
      </div>

      <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
        {/* Data Input Section */}
        <div className="bg-white rounded-lg shadow-lg p-6">
          <h2 className="text-xl font-semibold mb-4">Poll Configuration</h2>
          
          <div className="mb-4">
            <label className="block text-sm font-medium text-gray-700 mb-2">Poll Question</label>
            <input
              type="text"
              value={pollQuestion}
              onChange={(e) => setPollQuestion(e.target.value)}
              className="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
              placeholder="Enter your poll question"
            />
          </div>

          <div className="mb-4">
            <h3 className="text-lg font-medium mb-3">Response Options</h3>
            {responses.map((response, index) => (
              <div key={index} className="flex items-center gap-2 mb-2">
                <input
                  type="text"
                  value={response.option}
                  onChange={(e) => {
                    const updated = [...responses];
                    updated[index].option = e.target.value;
                    setResponses(updated);
                  }}
                  className="flex-1 p-2 border border-gray-300 rounded focus:ring-2 focus:ring-blue-500"
                />
                <input
                  type="number"
                  value={response.votes}
                  onChange={(e) => updateVotes(index, e.target.value)}
                  className="w-20 p-2 border border-gray-300 rounded focus:ring-2 focus:ring-blue-500"
                  min="0"
                />
                <button
                  onClick={() => removeResponse(index)}
                  className="p-2 text-red-600 hover:bg-red-50 rounded"
                >
                  <Trash2 size={16} />
                </button>
              </div>
            ))}
            
            <div className="flex items-center gap-2 mt-3">
              <input
                type="text"
                value={newOption}
                onChange={(e) => setNewOption(e.target.value)}
                placeholder="New option"
                className="flex-1 p-2 border border-gray-300 rounded focus:ring-2 focus:ring-blue-500"
              />
              <input
                type="number"
                value={newVotes}
                onChange={(e) => setNewVotes(e.target.value)}
                placeholder="Votes"
                className="w-20 p-2 border border-gray-300 rounded focus:ring-2 focus:ring-blue-500"
                min="0"
              />
              <button
                onClick={addResponse}
                className="p-2 bg-blue-600 text-white rounded hover:bg-blue-700"
              >
                <Plus size={16} />
              </button>
            </div>
          </div>
        </div>

        {/* Results Summary */}
        <div className="bg-white rounded-lg shadow-lg p-6">
          <h2 className="text-xl font-semibold mb-4">Results Summary</h2>
          
          <div className="mb-4">
            <div className="text-sm text-gray-600 mb-2">Total Votes: <span className="font-semibold">{analysisData.totalVotes}</span></div>
            <div className="text-sm text-gray-600 mb-4">Winner: <span className="font-semibold text-green-600">{analysisData.winner?.option}</span></div>
          </div>

          <div className="space-y-2">
            {analysisData.dataWithPercentages.map((item, index) => (
              <div key={index} className="flex justify-between items-center p-3 bg-gray-50 rounded">
                <span className="font-medium">{item.option}</span>
                <div className="text-right">
                  <div className="font-bold">{item.votes} votes</div>
                  <div className="text-sm text-gray-600">{item.percentage}%</div>
                </div>
              </div>
            ))}
          </div>

          <button
            onClick={exportReport}
            className="w-full mt-4 p-3 bg-green-600 text-white rounded-lg hover:bg-green-700 flex items-center justify-center gap-2"
          >
            <Download size={16} />
            Export Report
          </button>
        </div>
      </div>

      {/* Visualization Section */}
      <div className="bg-white rounded-lg shadow-lg p-6 mt-6">
        <div className="flex justify-between items-center mb-6">
          <h2 className="text-xl font-semibold">Data Visualization</h2>
          <div className="flex gap-2">
            <button
              onClick={() => setChartType('pie')}
              className={`p-2 rounded flex items-center gap-2 ${chartType === 'pie' ? 'bg-blue-600 text-white' : 'bg-gray-200 text-gray-700'}`}
            >
              <TrendingUp size={16} />
              Pie Chart
            </button>
            <button
              onClick={() => setChartType('bar')}
              className={`p-2 rounded flex items-center gap-2 ${chartType === 'bar' ? 'bg-blue-600 text-white' : 'bg-gray-200 text-gray-700'}`}
            >
              <BarChart3 size={16} />
              Bar Chart
            </button>
          </div>
        </div>

        <div className="h-96">
          <ResponsiveContainer width="100%" height="100%">
            {chartType === 'pie' ? (
              <PieChart>
                <Pie
                  data={analysisData.dataWithPercentages}
                  cx="50%"
                  cy="50%"
                  labelLine={false}
                  label={({ option, percentage }) => `${option}: ${percentage}%`}
                  outerRadius={80}
                  fill="#8884d8"
                  dataKey="votes"
                >
                  {analysisData.dataWithPercentages.map((entry, index) => (
                    <Cell key={`cell-${index}`} fill={colors[index % colors.length]} />
                  ))}
                </Pie>
                <Tooltip formatter={(value, name) => [`${value} votes`, 'Votes']} />
              </PieChart>
            ) : (
              <BarChart data={analysisData.dataWithPercentages}>
                <CartesianGrid strokeDasharray="3 3" />
                <XAxis dataKey="option" />
                <YAxis />
                <Tooltip formatter={(value, name) => [`${value} votes`, 'Votes']} />
                <Legend />
                <Bar dataKey="votes" fill="#8884d8" />
              </BarChart>
            )}
          </ResponsiveContainer>
        </div>
      </div>
    </div>
  );
};

export default PollAnalyzer;
